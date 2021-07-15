---
layout: post
title:  "jemalloc介绍"
categories: c/c++ 优化
tags:  netty
author: liubo
typora-root-url: ..
---

* content
{:toc}


## jemalloc基本信息

在学习Netty时，发现buffer引入缓冲池，其借用了jemalloc的思想；jemalloc在android5.1开始，成为libc默认的heap管理器。在这里搜索下jemalloc相关资料，以作记录。

jemalloc使用一系列算法保证程序在申请和释放内存的高效稳定，主要有以下几个特点：

1. 线程默认都有自己的私有缓存空间tcache，线程中分配内存都走这里，若未分配成功再走其他流程，线程内部的缓存空间中申请内存可以不加锁。

2. 内存区重新规划，使用不同大小的分配单元

3. 内存释放时，小内存合并为大内存，并判断是否需要缓存以供后续使用




## 其他内存管理工具

jemalloc facebook开源

tcmalloc google开源


## 参考资料

[jemalloc代码](https://github.com/jemalloc/jemalloc)

[tcmalloc代码](https://github.com/google/tcmalloc)

[jemalloc介绍](https://uncp.github.io/JeMalloc/)

[jemalloc内存管理](https://www.cnblogs.com/gaoxing/p/4253833.html)

[ptmalloc、tcmalloc和jemalloc对比](http://www.cnhalo.net/2016/06/13/memory-optimize/)