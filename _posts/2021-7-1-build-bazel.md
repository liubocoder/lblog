---
layout: post
title:  "Bazel构建工具"
categories: Build
tags:  Bazel Onos
author: liubo
typora-root-url: ..
---

* content
{:toc}


## 前言
Bazel是Google开源的构建工具，原生支持java、C++的编译，开源社区也有支持其他语言的编译的rule。本文结合了自身实践过程来学习bazel。

## Bazel安装
bazelisk 是bazel的版本管理工具，不同版本的onos使用了不同版本的bazel，因此我们在编译onos的时候bazelisk会自动下载对应版本的bazel进行编译。

```
wget https://github.com/bazelbuild/bazelisk/releases/download/v1.4.0/bazelisk-linux-amd64
```




![img](/assets/blog-img/build-bazel-1-1.png)

```
chmod +x bazelisk-linux-amd64
sudo mv bazelisk-linux-amd64 /usr/local/bin/bazel
```

![img](/assets/blog-img/build-bazel-1-2.png)

安装gcc
```
sudo apt install gcc
sudo apt install g++
```
当然也可使用各个linux版本各自的包管理器来安装，例如Ubuntu
```
sudo apt install bazel
```

## Bazel介绍

bazel以target作为构建的最小单元。一个项目里面必须要包含WORKSPACE文件，一个源码包下需要有BUILD文件来定义target（这两类文件都可以加个后缀.bazel，也能被识别）

先看一个测试工程

![img](/assets/blog-img/build-bazel-2-1.png)

FuncInterface是一个接口，FuncImpl实现该结构，MainTest包含main函数，调用FuncImpl的实现方法。

BUILD

![img](/assets/blog-img/build-bazel-2-2.png)

WORKSPACE为一个空文件

demo中的BUILD文件定义了一个java_binary类型的target

> - load：引入rule，bazel需要知道源码具体是如何编译，是通过rule完成
> - name：每一个target都有一个name，用于在其他地方被引用
> - src：需要编译的源文件
> - main_class：main函数class
> - glob：该函数是使用shell模式的通配符进行匹配

执行编译`bazel build :MyBinary`

![img](/assets/blog-img/build-bazel-2-3.png)

执行`bazel run :MyBinary`即可执行编译的main函数。

### BUILD

Bazel一个workspace下包含0到多个package，每个package包含0或多个target；其中target包括file、rule、packagegroup三种类型。

![img](/assets/blog-img/build-bazel-2-4.png)

工作区：Workspace
在项目跟路径下需要创建WORKSPACE(或WORKSPACE.bzl)文件，bazel在构建过程中创建一个隔离的工作区，用于标示项目的起始位置。

项目的源文件、构建过程的生成文件、及其所有外部依赖都归属该工作区。

WORKSPACE文件用于声明项目名称，及其该项目的外部依赖，如果项目不存在外部依赖，该文件内容可以为空（但不能没有文件）。

工作区作为隔离区，可以隔离不同版本的依赖。

### 依赖

本地依赖是@来标示，通常是可以省略，外部依赖是通过@name来标示，name是外部依赖的名称，这里测试下依赖远端仓库的jar包，修改WORKSPACE文件

![img](/assets/blog-img/build-bazel-2-5.png)

参数license_types是license-type字符串的一个列表，有效的license types包括：restricted、reciprocal、notice、permissive、unencumbered

在BUILD中添加deps，填写依赖关系，在代码中使用fastjson

![img](/assets/blog-img/build-bazel-2-6.png)

执行编译

![img](/assets/blog-img/build-bazel-2-7.png)

外部依赖还可以通过其他方式引入：http_repository、new_http_repository等

### Package
含有BUILD的目录即为一个包，是一个独立的存在，如果一个目录不存在BUILD，那么属于父目录的BUILD包范围，BUILD决定包的范围。

包名是起始于BUILD文件所在的目录。

包集合标示一组包，由package_group定义。使用包集合，可以很方便的将某个规则和可见性一并赋予给包集合，使得该包集合所包含的包都可以访问该规则。

### Target
一般target是包含文件（File）、规则（rule）、包集合（package group）三种基本类型

> - 文件：源文件和派生文件
> - 规则：规则是由输入、输出、动作组成，动作一般是隐式的，比如java_library无需显示指定javac -jar
> - 规则输入：源文件、也可是规则通过deps表示

目标名称被称之为标签，标签是由三个部分组成

![img](/assets/blog-img/build-bazel-2-8.png)

前两部分在很多场景下都是省略了，前面执行的`bazel run :MyBinary`也可以写成`bazel run @//:MyBinary`。

### DAG依赖图
我们将demo修改一下：mybin依赖mylib

使用如下指令查看依赖图

```
xdot <(bazel query --nohost_deps --noimplicit_deps 'deps(//mybin:mybin)' --output graph)
```

![img](/assets/blog-img/build-bazel-2-9.png)

![img](/assets/blog-img/build-bazel-2-10.png)

### 宏函数
bazel提供了灵活的扩展机制，用于自定义宏函数，

![img](/assets/blog-img/build-bazel-2-11.png)

这里相当于执行了ls -l BUILD > fout
而输出文件fout是在bazel-out/k8-fastbuild/bin/fout

![img](/assets/blog-img/build-bazel-2-12.png)

还通过def关键字定义一个公共的宏函数

![img](/assets/blog-img/build-bazel-2-13.png)

这个宏函数需要定义在.bzl 文件中，我这里创建了一个tool/printer.bzl，在需要使用的地方使用load函数加载，构建自己的宏函数

![img](/assets/blog-img/build-bazel-2-14.png)

再次执行自定义rule生成的target

![img](/assets/blog-img/build-bazel-2-15.png)

### Remote Cache
bazel比较重要的一个特性就是支持远端缓存

要能够缓存Bazel每个action的输出，我们就要一个server来实现remote cache，用于存储action的输出。这些输出实际上是一堆文件对应的hash值。

总体来说，我们需要满足三个前提：
- 设置一个server作为cache backend
- 配置Bazel build去使用cache
- Bazel版本要在0.10.0以上

bazel的缓存服务器三种方式实现：
- NGINX的WebDAV模块
- 开源的bazel-remote
- Google Cloud Storage

我们选择开源的方案，直接使用docker来完成。先下载docker镜像：
```
docker pull buchgr/bazel-remote-cache
```
![img](/assets/blog-img/build-bazel-2-16.png)

创建一个cache目录，并将权限修改成777
```
mkdir cache; chmod 777 -R cache
```

再执行docker容器创建指令
```
path=`pwd`;docker run -v $path/cache:/data -p 9090:8080 -p 9092:9092 buchgr/bazel-remote-cache
````
![img](/assets/blog-img/build-bazel-2-17.png)

在本地测试远端服务器状态
![img](/assets/blog-img/build-bazel-2-18.png)

本地编译的时候加如下参数

`--remote_cache=http://replace-with-your.host:port`

添加认证信息

`--remote_cache=http://user:pass@replace-with-your.host:port`

若不想使用远端cache，可以在构建时指定参数，或在target中指定no-cache参数：

`build --remote_upload_local_results=false`

```
java_library(
    name = "target",
    tags = ["no-cache"]
)
```

编译后再次查看远端服务器状态，已经产生一些缓存

![img](/assets/blog-img/build-bazel-2-19.png)

## 参考资料

<https://www.jianshu.com/p/3a4a2b5f46de>

<https://www.zhihu.com/people/thewarrior/posts>

<https://docs.bazel.build/versions/4.0.0/bazel-overview.html>

<https://github.com/buchgr/bazel-remote>