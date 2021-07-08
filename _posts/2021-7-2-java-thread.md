---
layout: post
title:  "Java线程相关"
categories: java
tags:  thread threadlock
author: liubo
typora-root-url: ..
---

* content
{:toc}


## 锁

1. 偏向锁/轻量级锁/重量级锁
是指synchronized锁状态，描述在对象头部Mark Word（可以使用库jol-core，ClassLayout.parseInstance()方法查看）

2. 可重入锁/非可重入锁
可重入锁代表ReentrantLock，reentrant意思是‘再次进入’

3. 共享锁/独占锁
ReadWritLock

4. 公平锁/非公平锁
synchronized 是非公平锁，ReentrantLock通过带参构造公平锁




5. 悲观锁/乐观锁
悲观锁是假定每次获取数据都认为数据会被更改，因此在操作数据前必须要拿到锁，而乐观锁在获取资源前不要求锁住资源，而利用CAS在不独占资源情况下更改资源。

6. 自旋锁/非自旋锁
自旋是陷入循环

7. 可中断锁/不可中断锁
使用ReentrantLock.lockInterruptibly()获取锁的过程是可被中断

## java 等待一个线程结束的方案

1. while循环

2. Object的wait/notify方法

3. 使用锁

4. 其他封装方法 如：CountDownLatch、CyclicBarrier、Thread.join()、Future等等，都是直接或间接使用锁或wait/notify机制