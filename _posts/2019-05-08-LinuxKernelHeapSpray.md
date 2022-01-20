---
title: Linux Kernel universal heap spray
tags: pwn
# article_header:
#   type: cover
#   image:
#     src: /assets/images/helloworld.jpg
--- 

<!-- write excerpt here -->
Linux Kernel universal heap spray

<!--more-->


by [Vitaly Nikolenko](https://twitter.com/vnik5287)


## Introduction

This kernel heap spraying technique was demonstrated during the [beVX workshop](https://www.beyondsecurity.com/bevxcon/bevx-training.html) DCCP UAF n-day and then used for the 0day in the kernel IrDA subsystem (Ubuntu 16.04). Unlike the existing public heap sprays, it is applicable to very small objects (under 8 or 16 bytes in size) or objects where we need the first N bytes to be controlled (i.e., no uncontrolled header in the target object). This techniques was used to exploit two UAF kernel vulnerabilities and as I've mentioned during the workshop, the term "heap spray" is not really applicable when it comes down to exploiting UAF vulnerabilities. It is generally not required to "spray" (allocate) multiple objects. A single properly-placed allocation is generally all that's needed to place the target object over the previous allocation of the freed/vulnerable object.

That being said, heap sprays (allocating multiple objects to fill in the slabs) definitely apply to other heap related vulnerabilities such as kernel heap overflows.

There's currently a lot of misinformation around public heap sprays such as `add_key()`, `send[m]msg()`, and `msgsnd()`. All these sprays have either object size restrictions or a header (as the first N bytes of the object) that cannot be controlled with user-space data. For example, the `msgsnd()` allocation path has a 48-byte "uncontrolled" header when kmalloc'ing the object.

The following user-space code will trigger the `do_msgsnd` execution path shown below.

```
#define BUFF_SIZE 96-48

struct {
	long mtype;
	char mtext[BUFF_SIZE];
} msg;

memset(msg.mtext, 0x42, BUFF_SIZE-1);
msg.mtext[BUFF_SIZE] = 0;
msg.mtype = 1;

int msqid = msgget(IPC_PRIVATE, 0644 | IPC_CREAT);

msgsnd(msqid, &msg, sizeof(msg.mtext), 0);
```

First the 96-byte object (48 byte `struct msg_msg` + 48 bytes for body of the message) is allocated in `load_msg()` in [1]:

```
long do_msgsnd(int msqid, long mtype, void __user *mtext,
                size_t msgsz, int msgflg)
{
        struct msg_queue *msq;
        struct msg_msg *msg;
        int err;
        struct ipc_namespace *ns;

        ns = current->nsproxy->ipc_ns;

        if (msgsz > ns->msg_ctlmax || (long) msgsz < 0 || msqid < 0)
                return -EINVAL;
        if (mtype < 1)
                return -EINVAL;

[1]     msg = load_msg(mtext, msgsz);
...
```

The user-space buffer content is then copied into the newly allocated kernel buffer:

```
struct msg_msg *load_msg(const void __user *src, size_t len)
{
        struct msg_msg *msg;
        struct msg_msgseg *seg;
        int err = -EFAULT;
        size_t alen;

        msg = alloc_msg(len);
        if (msg == NULL)
                return ERR_PTR(-ENOMEM);

        alen = min(len, DATALEN_MSG);
[2]     if (copy_from_user(msg + 1, src, alen))
                goto out_err;

        for (seg = msg->next; seg != NULL; seg = seg->next) {
                len -= alen;
                src = (char __user *)src + alen;
                alen = min(len, DATALEN_SEG);
                if (copy_from_user(seg + 1, src, alen))
                        goto out_err;
        }
...
```

The `alloc_msg()` implementation is shown below. The maximum message size is `DATALEN_MSG` (4048 bytes). If the message size is greater than `DATALEN_MSG`, then the message is considered to be multi-segment and the rest of the message is split into segments (8 bytes for the segment header + the rest of the message) [4].

```
static struct msg_msg *alloc_msg(size_t len)
{
        struct msg_msg *msg;
        struct msg_msgseg **pseg;
        size_t alen;

[3]     alen = min(len, DATALEN_MSG);
        msg = kmalloc(sizeof(*msg) + alen, GFP_KERNEL);
        if (msg == NULL)
                return NULL;

        msg->next = NULL;
        msg->security = NULL;

        len -= alen;
        pseg = &msg->next;
        while (len > 0) {
                struct msg_msgseg *seg;
                alen = min(len, DATALEN_SEG);
[4]             seg = kmalloc(sizeof(*seg) + alen, GFP_KERNEL);
                if (seg == NULL)
                        goto out_err;
                *pseg = seg;
                seg->next = NULL;
                pseg = &seg->next;
                len -= alen;
        }
...
```

Given there's always a 48-byte uncontrolled header, this spray is obviously not very useful for target objects in kmalloc-8, 16, 32 caches or any other target objects where we need to control the first 48 bytes (e.g., the function pointer is at offset 0 in the vulnerable object).

Ideally, the following conditions should hold for a universal heap spray:

1. Object size is controlled by the user. No restrictions even for very small objects (e.g., kmalloc-8).
2. Object contents is controlled by the user. No uncontrolled header at the beginning of the object.
3. The target object should "stay" in the kernel during the exploitation stage. This is especially useful for tricky UAFs and race conditions.

The `msgsnd` spray shown above satisfies only the third condition in this list. The good thing about this code path is that allocated objects remain in the kernel until the process either terminates or we call the `msgrcv` function to pop one of the messages from the queue.

There're also several paths in the kernel that satisfy the first two conditions but not the third one. By definition, these are often considered as sprays since they do spray the range with controlled user data. The only problem is that the object allocation path is immediately followed by the freeing path. When used with UAFs or race conditions, these "sprays" are not reliable (especially for objects in frequently used caches such as kmalloc-64). The other downside is that they cannot be used for target objects that require the first 4 or 8 bytes to be sprayed with user data. When the object is freed the first 8 bytes is overwritten with a slab freelist ptr value pointing to next free element in the slab (e.g., see this [counter overflow example](https://cyseclabs.com/blog/cve-2016-6187-heap-off-by-one-exploit)).

## userfaultfd + setxattr universal heap spray

The general approach is to take one of these kmalloc->kfree execution paths (conditions 1 and 2) and combine it with `userfaultfd` to satisfy the third condition.

`userfaultfd` has a good man page explaining its usage and providing examples on how to set up the page fault handler thread. The basic idea is to handle page faults in user space. For example, any lazy allocation in user space `testptr = mmap(0, 0x1000, ..., MAP_ANON|...)`followed a read or write access to the testptr page will trigger a page fault handler in kernel space. With `userfaultfd`, these page faults can be processed/resolved in user space by a separate thread.

The next step is to find a kernel execution path that satisfies conditions 1 and 2 described above. For example, the `setxattr()` syscall has the following kmalloc->kfree path:

```
static long
setxattr(struct dentry *d, const char __user *name, const void __user *value,
         size_t size, int flags)
{
        int error;
        void *kvalue = NULL;
        void *vvalue = NULL;    /* If non-NULL, we used vmalloc() */
        char kname[XATTR_NAME_MAX + 1];

        if (flags & ~(XATTR_CREATE|XATTR_REPLACE))
                return -EINVAL;

        error = strncpy_from_user(kname, name, sizeof(kname));
        if (error == 0 || error == sizeof(kname))
                error = -ERANGE;
        if (error < 0)
                return error;

        if (size) {
                if (size > XATTR_SIZE_MAX)
                        return -E2BIG;
[5]             kvalue = kmalloc(size, GFP_KERNEL | __GFP_NOWARN);
                if (!kvalue) {
                        vvalue = vmalloc(size);
                        if (!vvalue)
                                return -ENOMEM;
                        kvalue = vvalue;
                }
[6]             if (copy_from_user(kvalue, value, size)) {
                        error = -EFAULT;
                        goto out;
                }
                if ((strcmp(kname, XATTR_NAME_POSIX_ACL_ACCESS) == 0) ||
                    (strcmp(kname, XATTR_NAME_POSIX_ACL_DEFAULT) == 0))
                        posix_acl_fix_xattr_from_user(kvalue, size);
        }

        error = vfs_setxattr(d, kname, kvalue, size, flags);
out:
        if (vvalue)
                vfree(vvalue);
        else
[7]             kfree(kvalue);
        return error;
}
```

First, an object is allocated (size is user-controlled) [5] and then user-space data is copied to the allocated object in [6]. This path satisfies our heap spray conditions 1 and 2. However, the allocated `kvalue`pointer is then freed on the same execution path in [7].

The idea is to combine this execution path with a userfaultfd user-space buffer/page so that when the execution flow reaches [5], the page fault is handled in a user-space thread. This thread can then sleep indefinitely, effectively allowing the sprayed object to "stay" in kernel space. For example, let's assume the target object is 16 bytes and the first 8 bytes is the function pointer:

```
struct target {
	void (*fn)();
	unsigned long something;
};
```

The first step is to allocate two adjacent pages in user space and place the target object on the boundary of these two pages; the first 8 bytes (i.e., the function pointer) are on in the first page and the remaining bytes are placed in the second page. The next step is to use `usefaultfd` syscall to set up the page fault handler on the second page to handle all PFs in a user-space thread. Page faults for the first page are still handled by the kernel.

![userfaultfd2](userfaultfd2.svg)

When the kernel execution path in `setxattr()` gets to [6], the function pointer (8 bytes) will be copied to `kvalue` but any attempt to copy the remaining bytes will transfer execution to our user-space thread handling PFs for that page. The thread can then sleep for N seconds ensuring that the object stays in the kernel when the UAF path is triggered.

## Conclusion

This technique was used successfully during the workshop to exploit the recent DCCP UAF (target object size 16 bytes with first 8 bytes being the function pointer) and another 0day in the IrDA subsystem (56 bytes for the vulnerable object with the first 16 bytes being target pointers). There're other kernel execution paths (satisfying conditions 1 and 2 above) that can be combined with `userfaultfd()`. The only downside of this approach is that the main user-space execution path is suspended as well (when PF is handled in a separate thread). This can be solved by forking another process or creating a second thread (e.g., child process performs the heap spray and the parent process triggers the UAF).