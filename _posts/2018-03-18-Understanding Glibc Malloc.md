---
title: Understanding Glibc Malloc
description: 
categories:
 - tutorial
tags:
 - pwd
---

> For fast review

翻译自 https://sploitfun.wordpress.com/2015/02/10/understanding-glibc-malloc/

# Understanding Glibc Malloc
[TOC]

除了Glibc使用的ptmalloc，还有这些种类：
+ dlmalloc – General purpose 
+ allocator
+ ptmalloc2 – glibc
+ jemalloc – FreeBSD and Firefox
+ tcmalloc – Google
+ libumem – Solaris
+ …

ptmalloc2源自dlmalloc，加入了线程控制的功能。ptmalloc与glibc实际整合的malloc还是有一些区别
## System Call
malloc实际上都调用的是brk或者mmap系统调用

ptmalloc也是Linux的默认allocator（支持线程的原因）
## Threading
在ptmalloc2中，每个线程维护分开的堆部分，因此可利用空间表（freelist）也是相互隔离的，这种做法叫做per thread arena

## 第一次调用malloc
- 尽管用户可能只申请了1000bytes，但是还是会申请132k(Minsize)的连续堆空间。主线程新开辟出的这一段连续空间被称为main arena，若再继续allocate，则需要从这个arena中拿空间直到其空间耗尽。
- malloc实际上是调用brk的系统调用，将程序使用的break location增加。

当arena空间耗尽，再次系统调用，申请增加break location（这个发生在顶层的chunk已无法调整供申请的空间之后）

实际上，arena的实际使用空间可能会随着top chunk的过多离散空余空间而缩水

## free
在用户调用了free之后，空间并没有马上还给系统，而是在“glibc库”中释放，glibc将这块空间交给main arena bin
- 在glibc的malloc中，可利用空间表数据结构被称为bins

在用户再次申请空间时，就可以直接从bin中找空闲的block，只有当没有空闲的block时，才再次调用br

## 线程malloc
在进入到线程之后，堆空间不会被分配，但此时线程独立的栈空间已经被分配。

在线程中调用malloc之后，glibc调用mmap（与主线程使用sbrk不同）。尽管也只申请了1000bytes空间，但是却拿到了1M的内存空间，从中分配出132k的读写空间给用户用作堆，1M中剩余的空间设为不可读写，暂时留存。这132k的连续空间成为thread arena
- 但是当用户申请超过128k（即malloc(132*1024)）的空间时，没有arena能满足需要，因此无论是main还是thread arena都会调用mmap

同样当线程调用free时，将这1000字节的空间交给thread arenas bin

## Arena

### Number of arena
一些程序可能有超过cpu核个数的线程数，此时如果每个线程都分配独立的arena就显得有些低效，因此应用的arena个数限制基于系统的core个数:
```
For 32 bit systems:
    Number of arena = 2 *number of cores.
     
For 64 bit systems:
    Number of arena = 8 *number of cores.
```

### 多重arena
当arena个数大于上述的限制时，就需要在几个可能的线程之间进行arena的共享：（以32位系统3线程1核为例）

- 如果主线程第一次调用malloc已经创建了主arena
- 当线程1进行malloc，没有竞争，直接进行一对一分配
- 当线程2想malloc时，超过上限，因此尝试重新使用原有的arena(Main or thread 1)
    - 循环检查所有的arena，尝试给检查的arena上锁
    - 上锁成功，将这个arena return
    - 如果没有，暂停线程，等待空余arena
- 假设上述过程使thread2和main thread共用arena。则当thread2再次malloc时，会尝试使用最后accessed的arena，即main arena。如果main arena free了就直接占用，否则等待main arena free。
    - 即现在arena在main和thread 2 之间共用了

### 多重heap
在堆结构中有三个重要的数据结构：
- heap_info（Heap Header）
一个arena中有多个heap，每个heap都有自己的header。
    = 多个heap产生的原因：当一个heap segmeng空间耗尽时，新的heap分配不连续
- malloc_state（Arena Header）包含bins、top chunk、last remainder chunk...etc的信息
- malloc_chunk（chunk eheader）一个heap被分为多个chunk，每个chunk又有自己的header

主线程没有多重heap，因此也没有heap_info的结构，当main arena空间耗尽，就调用[sbrk](https://en.wikipedia.org/wiki/Sbrk)方法来扩充heap空间直到其到达内存中正在被使用的部分(Memory mapping segment)
主线程的arena header并不在主线程的heap段内，它在libc动态库中以全局变量的形式存在

![](http://r.photo.store.qq.com/psb?/V13dixb24fK0LX/ngLb4CAFEjfQ2e4iEbIkS1XPz7LBof776QXA6LNblz8!/r/dG0BAAAAAAAA)
Pictorial view of main arena and thread arena (single heap segment)
![](http://r.photo.store.qq.com/psb?/V13dixb24fK0LX/JYMfxtTTYVf.uSk3CQdyGknmi0nagf*Ksyd4d5eiE94!/r/dD0BAAAAAAAA)
Pictorial view of thread arena (multiple heap segment’s)

## Chunk

heap segment内的chunk有如下几种类型：

- Allocated chunk
- Free chunk
- Top chunk
- Last Remainder chunk

### Allocated chunk

图中的PMN为flag information：
- PREV_INUSE(P):– This bit is set when previous chunk is allocated.
- IS_MMAPPED(M)– This bit is set when chunk is mmap’d.
- NON_MAIN_ARENA(N)– This bit is set when this chunk belongs to a thread arena.

而其他一些如fd，bk并不是用来分配chunk的，因此一些用户数据储存在其中

用户申请的空间会被转变为实际使用的空间(Internal representation size)，因为有一些额外的空间要用来保存malloc_chunk和一些其他的安排信息。正是因为有了这样的安排，最后3个flag为才能留出来用作flag位

### Free chunk
不存在两个free chunk相邻的情况，因为一旦有，将两个合并。因此这个free chunk的前一个chunk一定是allocated的，因此free chunk的prev_size包含前一个chunk的用户数据

这里引入两个新变量：
- fd：Forward pointer – Points to next chunk in the same bin (and NOT to the next chunk present in physical memory).
- bk：Backward pointer – Points to previous chunk in the same bin (and NOT to the previous chunk present in physical memory).

## Bins
bins就是前文提到的可利用空间表，用来装载free chunks。根据chunk大小不同，分为下列4种bin：
- Fast bin （单链表结构）
- Unsorted bin（双链表结构）
- Small bin（双链表）
- Large bin（双链表）
用来处理bins的数据结构包括：
- fastbinsY:用来保存fast bins的数组
- bins：用来保存其它3类bins的数组。数组大小为126，分为以下3种：
    - bin 1:Unsorted bin
    - bin 2-63:Small bin
    - bin 64-126:Large bin

### Fast bin
Chunk size在16-80字节之间

fast bin有更好的内存分配和释放效率

由于增删操作都发生在array的头和尾，因此Fast bin采用单链表结构(LIFO后进先出原则)

fast bins可有多达10个

任一同一个fast bin中的chunk大小相同

不同fast bin之间的chunk均以8字节为大小间隔

在malloc初始化的过程中，默认fast chunk最大为64字节而不是80字节

即便两个free chunk相邻，也不会将其合并，这虽然会导致外部空间碎片化，但是提速。

malloc大小小于max_fastbin时从fastbin的前面拿一个出来

#### malloc(fast chunk)
最开始fast bin max size和fast bin indices都是空，因此即便用户请求一个fast chunk，也会由small chunk的处理程序来尝试处理它

当其不为空后，才能开始接受用户的fast bin请求，从队列头拿出一个chunk给user（LIFO）
#### free(fast chunk)
fast bin的下标进行重新计算
被释放的chunk加在此fast bin的前端

### Unsorted bin
当small或large bin被释放且没有被加入到相应的bins中时，就将其加入到unsorted bin中。

这样，glibc malloc就有了第二次机会来重用最近被释放的chunk

unsorted bins只有一个，双链表结构，任意大小的chunk都能放在里面

### Small bin
小于512字节的chunk均称为small chunk

Small bin处理速度比large bin快，比fast bin慢

Small bin数组大小最多为62，相邻间隔也是8bytes

与fast bin不同，它会对相邻的small chunk进行合并以减小碎片化，但是减慢了free的速度

#### malloc(small chunk)
与fast bin相似，在small bin初始化完成之前，所有请求将由unsorted bin方法尝试解决

在第一次调用malloc时，small和large bin的数据结构才会进行初始化

bin可能指向自身，此时表示自己为空

当small bin不为空，就从队列尾拿出一个chunk给user
#### free(small chunk)
释放时，首先检查之前的chunk是否被释放，如果是，就合并。将它们从相应的list中撤出，合并后加到unsorted bin list的开始

### Large bin
大于512字节的chunk称为large ~

处理速度比small bin慢

bin的数组大小为63，使用循环双向链表（因为可能从中任意位置抽出或加入chunk）
- 32个bin保存大小从512字节往后间隔为64字节的chunk
- 16个保存间隔大小为512字节的
- 8个保存间隔4096bytes
- 4~32768
- 2~262144
- 最后1个保存一个剩余空间大小的chunk

每个large bin中的chunk大小不一定相同，因此会将其排序。最大的放最前面，最小的放最后。

会对相邻的chunk做合并
#### malloc(large chunk)
当前如果large bin为空，则由下一个最大的large bin对应的代码方法来尝试处理。

初始化时间与small bin相同，指向自身时同样表示为空

如果bin list不为空，且最大的chunk大于用户需要的大小，就从list的最后（即最小的chunk）开始从后往前找到一个chunk大小与用户需求相近的chunk
- 这个chunk会被分为2部分：
    - User chunk交给用户
    - Remainder chunk加到unsorted bin中

如果最大的不满足，就尝试使用下一个更大的binlist来解决。轮询之后若还没有，就尝试用top chunk来解决 
#### free(large chunk)
与free(small chunk)相似

## Top Chunk
arena顶部的边界chunk称为Top Chunk，其不属于任何bin，它用来满足在没有free chunk的情况下的用户请求。

如果它的大小大于用户请求，则按large bin方式分裂chunk（Remainder成为新的top chunk）；否则调用sbrk(main arena)或mmap(thread arena)
## Last Remainder Chunk
即最近一次small request中割出的剩余部分，能够增强定位引用的效率（连续的对small chunk的malloc申请会使small chunks最后都紧挨分配）

只在small chunk requeset 最后通过分裂满足后得到的remainder叫做last remainder chunk

为什么会有上述的内存放置连续性的现象：

>Now when user subsequently request’s a small chunk and if the last remainder chunk is the only chunk in unsorted bin, last remainder chunk is split into two, user chunk gets returned to the user and remainder chunk gets added to the unsorted bin. In addition to it, it becomes the new last remainder chunk. Thus subsequent memory allocations end up being next to each other.

最后附神图：malloc/free/realloc的系统处理过程
![](http://r.photo.store.qq.com/psb?/V13dixb24fK0LX/4O76AgCni1jpxJtU3GtvIeOcA.2hT8Eq1BsfhWKrzaw!/r/dOcAAAAAAAAA) 

1
