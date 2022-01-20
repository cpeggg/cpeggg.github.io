---
title: _IO_FILE Structure
description: a basic description of file structure in glibc
tags:
 - pwn
---
#_IO_FILE structure
应该知道的是，该structure是[heap]段中malloc出的一个chunk，而且该structure中指向文件内容的指针所指向的地址也在[heap]段中，为0x1000(0x1010)大小的chunk
文件读写常用的可打开的特殊文件：（ongoing）
- /dev/stdin (/dev/fd/0)
- /dev/null
对一些特殊FILE structure的修改：
- 对stdin的buf_end的修改可以造成libc内的“堆”溢出，造成大量全局变量的修改，或者造成unsorted bin attack，甚至可以一直写到main_arena
- 诸如put/printf/scanf/scanf/fgets/gets也是需要使用到stdin/stdout/stderr等file结构体的

Windows下的pwn：
FILE没有vtable，但是仍然有buffer的指针，仍可以造成任意读写
![@glibc-2.24 libio.h](./1511313827739.png)
![@glibc-2.24 libioP.h](./1511314236933.png)


