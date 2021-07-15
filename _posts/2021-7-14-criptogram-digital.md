---
layout: post
title:  "数字签名"
categories: 加密算法
tags:  digital
author: liubo
typora-root-url: ..
---

* content
{:toc}


## 概念
数字签名又名公钥数字签名，基于非对称加密算法和数字摘要算法，具有三大特性：可认证性(authentication)、不可抵赖性(non-repudiation)和完整性(integrity)。

首先我们考虑一个场景，客户端B在访问一个域名服务器S时要解决几个问题
 > 1. 保证数据的安全，不被窃听  
 > 2. 保证数据的完整，不被篡改  
 > 3. 保证服务器是合法的，可认证的

其中最主要的是保证服务器的合法性，其他的都可以在确保服务器合法后通过协商密钥加密传输数据来解决，而保证服务器合法性的根本是证书，证书的基础就是数字签名。

当然证书是验证证书持有者的合法性，所以不仅能验证服务器也能验证客户端，例如：银行的U盾用于验证客户端的合法性。

数字签名的核心还是使用的是非对称加密算法的私钥对需要签名的数据进行加密处理。




## 存储
数字签名中牵扯到密钥、证书、密钥库的存储，密钥和证书一般都是使用ASN.1语法记录，存储格式可以是DER这类二进制文件，也可以是PEM这种经过Base64编码后的字符文件。

常见证书文件后缀名有.cer、.crt、.rsa、.pem、.crs、.p7b等，windows系统能识别到.cer .crt .p7b的文件。

密钥库是包含公钥私钥和证书，其格式一般有JKS、JCEKS、PKCS#12等
 > JKS 扩展名.jks/.ks  
 > JCEKS 扩展名.jce  
 > PKCS#12 扩展名 .p12/.pfx

## 证书
国际通用x509标准来描述证书的基本结构，目前版本是v3，其规定证书有哪些字段，各自意义是什么，哪些是必要字段，哪些是可选字段。

前面提到证书是私钥对另外一个公钥的签名，而签名的私钥对应的公钥证书谁来颁发呢，所以引入了CA的概念，系统默认CA的自签名证书是合法的证书，目前操作系统中基本都安装了受信任的证书列表。

### SSL证书
若需要支持https，则需要对域名申请ssl证书。阿里云上可注册免费域名，我随便注册了一个liuliu.work，申请免费的SSL证书，根据不同的服务器类型提供下载各类的密钥库文件或证书（证书都是一样的，只是保存证书的格式区别）。

![img](/assets/blog-img/criptogram-digital-3-1.png)

证书中的CN字段就是申请的二级域名。

### 自签名证书
我们自己创建一个非对称加密密钥对，用私钥对公钥签发证书，这个证书就是自签名证书，其实和CA颁发的证书一样，只是自签名证书没有在系统的受信任证书列表中。

Android的APK中使用的就是自签名证书。

在Chrome设置中搜索证书，打开系统的证书管理页面，在这里可以导入自己的证书或证书库

![img](/assets/blog-img/criptogram-digital-3-2.png)

## SSL

## 相关代码

openssl
keytool

创建密钥对
创建密钥
签发证书

## 参考资料

 [Android数字签名机制](https://duanqz.github.io/2017-09-01-Android-Digital-Signature#13-%E4%BD%BF%E7%94%A8)

[PCKS标准](https://www.iteye.com/blog/falchion-1472453)

[keystore工具](http://keystore-explorer.org/)

<https://blog.csdn.net/zzhongcy/article/details/22755317>