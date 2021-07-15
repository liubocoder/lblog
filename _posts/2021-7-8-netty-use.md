---
layout: post
title:  "Netty框架"
categories: java
tags:  netty socket
author: liubo
typora-root-url: ..
---

* content
{:toc}


## 前言
在讨论Netty之前需要了解下我们处理网络请求的方案

1. BIO
同步阻塞，服务器调用accept阻塞等待请求，当请求到来时创建线程处理，主线程继续等待请求。

2. NIO
异步非阻塞，ServerSocketChannel非阻塞模式，使用selector轮询各类事件。

3. AIO
只需要关心buffer的读写事件

上述方案使用原生API直接开发，流程比较繁琐，性能不好，并且容易出错，使用Netty可以简化流程，Netty中使用Handler串行执行，结构层次分明，逻辑清晰，并且一个客户端只注册在一个线程上，避免部分业务中出现并发问题。




## 使用

ServerBootstrap/Bootstrap EventLoopGroup ChannelPipeLine ChannelHandler

### ServerBootstrap/Bootstrap

其中ServerBootstrap主要是用于服务器端，可以配置两个EventLoopGroup，分别用于接收请求和处理请求形象比喻boss-worker概念

### EventLoopGroup

使用某种IO复用策略的线程池，常用EpollEventLoopGroup、NioEventLoopGroup

### ChannelPipeLine

用于管理连接，编排Handler的调用顺序及事件。

新连接都会回调initChannel()方法

### ChannelHandler

ChannelHandler是实现业务中最重要的环节，常见的类型有：ChannelInboundHandler、ChannelOutboundhandler、ChannelDuplexHandler。

网络事件到来时，ChannelInboundHandler处理接收数据，ChannelOutboundhandler处理发送数据，而ChannelDuplexHandler两者都会处理

![img](/assets/blog-img/netty-use-1-1.png)

[图片出处](https://zhuanlan.zhihu.com/p/68061038)

业务中我们主要是和Handler打交道，常用Handler有

1. `SslHandler` 支持ssl传输加密
2. `IdleStateHandler` 支持超时检查，可用于实现长连接，包括read、write、all
3. 用于解决TCP粘包问题

    > `DelimiterBasedFrameDecoder` 分隔符解码器  
    `LineBasedFrameDecoder`  回车换行解码器  
    `FixedLengthFrameDecoder` 固定长度解码器  
    `LengthFieldBasedFrameDecoder` 用于标识消息体或者整包消息的长度，解决'读半包'

4. `FlushConsolidationHandler` 内置的flush优化器
5. 其他Handler

    > `LogginHandler` 日志打印  
    `StringEncoder`、`StringDecoder`、`ByteToMessageDecoder`、`DatagramPckagetDecoder`等等

#### 自定义handler
一般从ByteToMessageDecoder、ChannelOutboundHandlerAdapter、ChannelInboundHandlerAdapter等这类开始继承实现相关自定义业务。

通常业务中比较关心的是ChannelInboundHandler接口中的channelActive与channelInactive方法，连接建立与断开时的资源创建与释放等操作。

### ChannelOption常用配置

1. SO_KEEPALIVE

    > 是否保持长连接状态  
    默认false，表示关闭长连接

2. TCP_NODELAY

    > 是否禁用Nagle算法。Nagel算法将小的碎片数据连接成更大的报文来最小化所发送的报文的数量，如果需要发送一些较小的报文，则需要禁用该算法，从而降低报文传输延时  
    默认false，表示启用了该算法

3. SO_REUSEADDR

    > 是否复用处于TIME_WAIT状态连接的端口，适用于有大量处于TIME_WAIT状态连接的场景，如高并发量的Http短连接场景等  
    默认false，默认不复用

4. SO_LINGER

    > Socket关闭的延迟时间，socket.close() 方法调用后立即返回  
    默认-1，表示禁用该功能

5. SO_SNDBUF

    > TCP发送缓存容量最大值，即TCP发送滑动窗口，使用`cat /proc/sys/net/ipv4/tcp_smem`查询其大小  
    常用计算公式 缓冲区大小 = 网络带宽 * 网络时延

6. SO_RCVBUF
    > TCP接收缓存容量最大值  
    常用计算公式 缓冲区大小 = 网络带宽 * 网络时延

7. SO_BACKLOG

    > 对应的是tcp/ip协议, listen函数中的`backlog`参数，用来初始化服务端可连接队列。  
    如果backlog过小，就可能出现Accept的速度跟不上，A,B队列满了，就会导致客户端无法建立连接

## 参考资料

[Netty代码地址](https://github.com/netty/netty.git)

<https://blog.csdn.net/qq_18603599/article/details/80768390>