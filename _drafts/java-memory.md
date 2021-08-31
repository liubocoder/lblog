PC上的JVM实现版本主要有三类
Oracle HotSpot, Oracle JRockit, IBM JVM
移动平台Android的Dalvik/ART

我们最为关心的是JVM的内存模型，关系到业务运行的稳定性。
以下基于Hotspot虚拟机

##JVM内存模型

![img](/assets/blog-img/java-memory-1-1.png)
[图片出处]<https://imgconvert.csdnimg.cn/aHR0cHM6Ly9pbWcyMDE4LmNuYmxvZ3MuY29tL2Jsb2cvMTQ4OTY2OS8yMDE4MTAvMTQ4OTY2OS0yMDE4MTAwOTE4NTUyNzMxNi0xNzA4NzkwOTc0LnBuZw?x-oss-process=image/format,png>

1. 方法区
线程共享区域，存储已被虚拟机加载的类相关信息、常量、静态变量、即时编译器编译后的代码等。

2. 堆
线程共享区域，存储对象。
    1) gc能自动管理到的区域是java堆，java堆之外的内存可以通过以下方式申请到：
    ByteBuffer.allocateDirect(10*1024*1024)分配10M的内存空间。
    -XX:MaxDirectMemorySize=512m 最大值受该参数控制，

    2) jni通过C语言调用malloc分配，这种方式分配不再受JVM的参数约束，可以分配到操作系统允许进程占用内存的最大值，内存不够时会分配失败。

    注意：堆外内存需要主动释放，否则GC不会回收这部分内存。

3. 虚拟机栈
线程私有区域，java行的内存模型，存储局部变量表、操作栈、动态链接、方法出口等信息。
    可以优化虚拟机参数，使满足条件的对象（线程私有对象）存放在栈上。

4. 本地方法栈
为虚拟机使用到Native方法服务。

5. 程序计数器
线程私有，唯一没有定义OOM的区域。程序计数器是一块较小的内存空间，可以看作是当前线程所执行的字节码的行号指示器。分支、循环、跳转、异常处理、线程恢复等基础功能都需要依赖这个计数器来完成。

##GC
判断对象是可被回收的依据是“是否可达”，可达的根节点被称为GCROOTs，我理解的是方法区、虚拟机栈、本地方法栈持有了引用即可称为GCROOTs。
GC的目标是对象，对象的持有者是引用，java中引用有四类
强引用 FinalReference
软引用 SoftReference
弱引用 WeakReference
虚引用 PhantomReference
除强引用是我们不能操作的之外，其他三类引用我们都可以使用到，一般可以结合ReferenceQueue使用，判断该引用isEnqueued()是否入队，表示已被gc掉。
    Hotspot的收集器有3种
    1) 单线程 serial collector
    2) 并行 parallel collector
    3) 并发 concurrent collector
    [详情参考](https://www.cnblogs.com/redcreen/archive/2011/05/04/2037029.html)
    HotspotGC操作类型
    1) YGC eden空间不足
    2) FGC old、perm等空间不足  可以由System.gc()触发

##JVM参数
    JVM参数分为3类
    1) 标准参数：-version -showversion 等
    2) X参数：
        -Xms<大小>        设置初始 Java 堆大小，数字+m或g
        -Xmx<大小>        设置最大 Java 堆大小，数字+m或g
        ...
    3) XX参数：bool使用'+'表示值true，'-'表示值false，其他用'='连接具体值
        -XX:+DisableExplicitGC  禁止主动调用System.gc()
        -XX:+PrintGCDetails     打印gc信息
        -XX:NewSize=35m         新生代空间35m
        -XX:+PrintGCDetails     打印gc详情
        ...

[一些参数说明](https://www.cnblogs.com/redcreen/archive/2011/05/04/2037057.html)
[jvm参数配置](https://www.cnblogs.com/jack204/archive/2012/07/02/2572932.html)

    java打印运行参数
    System.getProperties(System.out)

##JVM调试
jdk自带的调试工具一般是在jdk的bin目录下，常用的工具：
jps 输出当前运行的虚拟机
jstat
jmap
jhat
jstack
jinfo

可视化工具
jconsole
jvisualvm


[jvisualvm可视化工具使用](https://www.cnblogs.com/jhxxb/p/13279201.html)


##JNI

<https://www.cnblogs.com/lovesaber/archive/2013/04/01/2993375.html>

注意查看自己的jvm库的位置，我的是在/usr/lib/jvm/java-8-openjdk-amd64
gcc -fPIC -I/usr/lib/jvm/java-8-openjdk-amd64/include -I/usr/lib/jvm/java-8-openjdk-amd64/include/linux -shared Mj.c -o libMj.so

java中加载动态库时注意java.library.path参数，如果不清楚可以用绝对路径来加载System.load()，当然用相对路径更好点System.loadLibrary()，相对路径的库命令为libxx.so。



