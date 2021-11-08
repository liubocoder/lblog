
#网络
##OAuth2.0协议理解
目的是拿到授权服务器的token，从而能在该服务器获取一些用户信息
[OAuth2.0 授权的4总方式](https://www.ruanyifeng.com/blog/2019/04/oauth-grant-types.html)

[]()

##TCP UDP区别， TCP性能为啥比UDP差， TCP为什么是三次握手， 三次握手有什么缺陷，四次挥手
四次挥手注意同时发起FIN的情况，双端都进入TIME_WAIT状态
[TCP连接建立和断开流程](https://www.cnblogs.com/onesea/p/13053697.html)
[理解TIME_WAIT](https://www.cnblogs.com/zhenbianshu/p/10637964.html)
TIME_WAIT等待的是MSL（最大段寿命，MAXimum Segment Lifetime ），不同主机上定义不同，标准是2分钟（RFC）
SYN同步（建立连接） ACK应答 FIN关闭  RST复位（出现异常，重置连接）

#数据结构和算法
##LinkedHashMap和HashMap

##二叉树的遍历方式有哪些，按深度优先和广度优先分类
前中后序遍历（注意递归思想，也就是数据结构-栈的思想），层序遍历（数据结构-队列的思想）
二叉树两个节点的最近祖先节点（可以分情况讨论，也可以直接遍历从根到目标节点的链表，然后对比两个链表）



#项目

##不同模块的通信方式
##minSdkVersion、targetSdkVersion作用，以什么驱动这两个版本的改变

##protobuf
[protobuf与JSON和XML比较](https://www.jianshu.com/p/a24c88c0526a)
[android中使用protobuf](https://www.jianshu.com/p/0f047e1b7e16)

##Binder机制
[Binder AIDL](https://blog.csdn.net/zizidemenghanxiao/article/details/50341773)