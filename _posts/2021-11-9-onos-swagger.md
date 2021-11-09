---
layout: post
title:  "记录ONOS自动添加的swagger文档中文乱码问题"
categories: build
tags:  bazel onos
author: liubo
typora-root-url: ..
---

* content
{:toc}


## 问题

在使用onos的doc时，添加中文的注释会在swagger页面显示乱码，原因是自动生成的swagger配置文件中写入了错误编码格式的字符。
![img](/assets/blog-img/build-onos-swagger-1-1.png)



## 定位问题原因

查阅onos源码，发现该配置文件是项目中一个工具模块在编译时自动生成，路径/yourproject/tools/build/swagger

![img](/assets/blog-img/build-onos-swagger-2-1.png)

源码是使用qdox工具，将接口类中的注释信息读取出来，按照既定的格式解析生成swagger.json配置文件，我们能确定java文件肯定是UTF-8格式，
因此只有执行时读取中文的时候jvm字符集不兼容导致的（排除javac，javac不识别的字符编码会直接编译出错）

![img](/assets/blog-img/build-onos-swagger-2-2.png)

为了再次确认，我在SwaggerGenerator.java中添加打印，打印当前虚拟机的默认字符集`Charset.defaultCharset()`，发现默认字符集是
US-ASCII，再次印证上面的推断。

## 解决问题

知道问题原因后，需要查看执行这段代码的地方在哪儿，在/yourproject/tools/build/bazel/osgi_java_library.bzl文件中，一个自定义rule

![img](/assets/blog-img/build-onos-swagger-3-1.png)

在命令实现的地方_swagger_json_impl，ctx.actions.run添加环境变量`env = {"_JAVA_OPTIONS":"-Dfile.encoding=UTF-8"}`，如下图所示

![img](/assets/blog-img/build-onos-swagger-3-2.png)

再次编译后，可以看到又出现可爱的中文了 :)

![img](/assets/blog-img/build-onos-swagger-3-3.png)