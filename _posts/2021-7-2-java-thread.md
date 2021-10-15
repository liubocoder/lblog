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

java中锁的实现分为两种：一种是synchronized关键字，一种是AbstractQueuedSynchronizer类，锁的核心就是如何管理竞争资源的线程。

当然根据实际的锁的业务类型，分为如下几类。

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

## 守护线程

Thread.setDeamon(boolean) 可将线程设置成守护线程，线程分为用户线程和守护线程，当不存在用户线程时，jvm会自动退出。

## 线程池

线程池一般是指实现ExecutorService接口，用于异步执行某个任务，可以通过Executors类静态方法构建线程池对象。
Executors提供以下方法构建线程池
newCachedThreadPool
    构造ThreadPoolExecutor对象，默认存活线程数量0，最大线程数量不限制，线程保活时长60s，队列使用SynchronousQueue

    SynchronousQueue 是一个生产者/消费者模型的队列实现，该队列没有容器，通过put确定产品生产者，通过take产品消费者，当消费不及时会阻塞生产，可以通过构造方法确定产品者是用队列还是栈来管理

newFixedThreadPool
    构造ThreadPoolExecutor对象，可设置固定线程数量，线程执行完后不会销毁


newScheduledThreadPool
    构造ScheduledThreadPoolExecutor对象，可设置核心线程数量，不限制最大线程数量，使用DelayedWorkQueue队列，可以延时执行任务

newSingleThreadExecutor
    线程池中只有一个线程，使用队列完成任务管理
newSingleThreadScheduledExecutor
    线程池中只有一个线程，使用队列完成任务管理，可延时执行任务
newWorkStealingPool
    构造ForkJoinPool对象，无序执行任务