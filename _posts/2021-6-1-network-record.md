---
layout: post
title:  "记录一些网络知识"
categories: 网络
author: liubo
typora-root-url: ..
---

* content
{:toc}


## 抓包

1. linux tcpdump
```
    示例：
    tcpdump -D 列举端口信息
    tcpdump -i eth0 -nn -XX  ip xx and port 123
    -i eth0 网卡eth0
    -nn 所有报文转数字
    -XX输出包的头部数据，会以16进制和ASCII两种方式同时输出，更详细
```




2. gui工具

    gui的抓包工具可以用wireshark，是一款非常牛的开源软件，默默念一句感谢开源世界。

## 路由相关

路由是网络层上的概念，指从某个接口出发到另外一个接口的路径描述，以下对linux和windows上的路由命令简单使用。

默认路由的概念

当目标主机的 IP 地址或网络不在路由表中时，数据包就被发送到默认路由（默认网关）上。默认路由的 Destination 是 default 或 0.0.0.0。

### linux路由配置

linux路由主要是route命令的使用

列举路由
```
route -n
```
添加路由
```
route add -net 10.0.0.10 netmask 255.255.255.255 gw 10.139.128.1 dev eth0
```
删除路由
```
route del -net 224.0.0.0 netmask 240.0.0.0
```

### windows路由配置
```
示例：
打印路由
route print
打印接口列表
route print -4

添加路由  -p属性永久添加
route add 0.0.0.0 mask 0.0.0.0 10.0.0.1 metric 3 if 12
    metric 跳数，约小优先级约高
    if 接口，使用route print -4查看

修改默认路由的网关为10.0.0.1，只能修改网关和跃点数
route change 0.0.0.0 mask 0.0.0.0 10.0.0.1

删除网关为10.0.0.1的默认路由
route delete 0.0.0.0 10.0.0.1

windows网卡的跃点数一般配置的自动跃点数，可以在网卡属性ipv4或ipv6的高级属性中修改成自定义的跃点数
```

## opennpv客户端启动后会修改默认路由
![img](/assets/blog-img/net-route-1-1.png)

将不认识的ip报文都转到tun0，内部再转向vpn服务器地址，从enp1s0网卡发出

## SSL
安全套接字层，一般是和http结合成https
区别SSH，SSH是由服务端和客户端组成连接终端，主要用于执行RPC，SSH基于SSL保障通信的安全。

## wifi wpa2-psk握手过程
<https://blog.csdn.net/Mr_gui_/article/details/113403996>
有机会再自己试试看

## 一些网络工具
<https://www.163.com/dy/article/FBNQV4350511RU8P.html>

## 参考资料

[tcpdump资料](https://www.cnblogs.com/f-ck-need-u/p/7064286.html)

[linux路由](https://blog.csdn.net/kikajack/article/details/80457841)