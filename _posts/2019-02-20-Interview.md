---
title: 计算机面试基础
tags: interview
# article_header:
#   type: cover
#   image:
#     src: /assets/images/helloworld.jpg
--- 

<!-- write excerpt here -->
自己在准备安全岗位面试时的一些其他的计算机基础知识

<!--more-->

# 一些比较好的链接🔗

- https://interview.huihut.com/#

# 操作系统

## 其他一些问题

### fork clone vfork区别

>     clone man page:
>     
>     Unlike fork(2), clone() allows the child process to share parts of its execution context with the calling process, such as the virtual address space, the table of file descriptors, and the table of signal handlers. (Note that on this manual page, "call‐ing process" normally corresponds to "parent process". But see the description of CLONE_PARENT below.)

fork和vfork的差别：

- fork 是 创建一个子进程，并把父进程的内存数据copy到子进程中。
- vfork是 创建一个子进程，并和父进程的内存数据share一起用。

这两个的差别是，一个是copy，一个是share。（关于fork，可以参看酷壳之前的《[一道fork的面试题](http://coolshell.cn/articles/7965.html)》）

你 man vfork 一下，你可以看到，vfork是这样的工作的，

1）保证子进程先执行。
2）当子进程调用exit()或exec()后，父进程往下执行。

### 进程间通信方式

临界区

互斥量

信号量（counter，PV操作）

事件

### 数据通信方式

（1）消息通信：类似于电子邮件系统。可以自己解决互斥与同步。

（2）共享存储区：通信速度快。需要借助额外的互斥与同步机制。

（3）管道通信：采用FIFO方式组织信息。可以实现大量信息交换。可以自己解决互斥与同步。

### 资源分配

银行家算法：[操作系统](https://baike.baidu.com/item/%E6%93%8D%E4%BD%9C%E7%B3%BB%E7%BB%9F)按照银行家制定的规则为进程分配资源，当进程首次申请资源时，要测试该进程对资源的最大需求量，如果系统现存的资源可以满足它的最大需求量则按当前的申请量分配资源，否则就推迟分配。当进程在执行中继续申请资源时，先测试该进程本次申请的资源数是否超过了该资源所剩余的总量。若超过则拒绝分配资源，若能满足则按当前的申请量分配资源，否则也要推迟分配。

### （不）可重入

主 要用于多任务环境中，一个可重入的函数简单来说就是可以被中断的函数，也就是说，可以在这个函数执行的任何时刻中断它，转入OS调度下去执行另外一段代 码，而返回控制时不会出现什么错误；而不可重入的函数由于使用了一些系统资源，比如全局变量区，中断向量表等，所以它如果被中断的话，可能会出现问题，这 类函数是不能运行在多任务环境下的。

#### 页表（逻辑地址->线性地址->物理地址）

我认为的过程应该是这样的：CPU首先访问某个虚拟地址上的数据，然后CPU中的段寄存器会根据CPU访问的数据类型（代码、数据等）返回一个段表的index，CPU找到这个段表项的信息（段基址、段长度等），确定是否访问越界。

如果没有越界，则根据段基址加上段内偏移得到线性地址，这个地址应该也就是cache所使用的地址。

然后如果cache miss，则把线性地址发送给MMU，MMU根据页表结构将线性地址转化为物理地址（注意页表的目录表的基地址存放在CR3寄存器当中），访问真正的物理内存

CPU产生逻辑地址，逻辑地址分为页表地址和页内偏移。而每个页对应物理内存中的一块，因此最终查到对应的页表项时能够得到对应的物理内存的块号，再根据页内偏移（块内偏移）就可以找到对应的数据了

