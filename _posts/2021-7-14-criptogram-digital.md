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
SSL（Secure Sockets Layer）一般是工作在传输层之上，用以保障应用层协议安全。SSL是基于证书、各类加密算法、数据摘要算法等各类算法之上，目前最新版本是TLS1.3版本，互联网标准化组织接手SSL后命名为TLS（Transport Layer Security）。

SSL位于传输层和应用层之间，应用层协议都可以基于SSL来保障数据安全。

## 相关代码

### keytool自建根证书及签发证书流程

1. 创建密钥仓库  
`keytool -genkey -alias rootali -keysize 2048 -validity 3650 -keyalg RSA -dname "CN=cn.lb" -keypass 123456 -storepass 123456 -keystore myrootStore.p12 -storetype PKCS12`  
`-storetype PKCS12`用于指定密钥库类型为PKC12，若不指定，默认是JKS  

2. 根据密钥仓库的某个密钥对 创建自签名证书  
`keytool -export -alias rootali -keystore myrootStore.p12 -storepass 123456 -file myCaCert.cer`

3. 创建受信任的证书仓库，这里直接导入了上一步自签名证书  
`keytool -importcert -alias mytrustalias -file myCaCert.cer -storetype PKCS12 -keystore mytruststore.p12`

4. 用户申请签发证书  
(1) 用户需要签发证书，首先得有个密钥对，所以先创建密钥对，与步骤1一致  
`keytool -genkey -alias userali1 -keysize 2048 -validity 365 -keyalg RSA -dname "CN=cn.lb.a" -keypass 123456 -storepass 123456 -keystore user1.p12 -storetype PKCS12`  
(2) 用户提出申请，生成证书签发请求文件  
`keytool -certreq -alias userali1 -file usercsr.csr -keystore user1.p12`

5. 签发证书
使用创建的根密钥库对用户的证书申请文件签名  
`keytool -gencert -validity 3650 -alias rootali -keystore myrootStore.p12 -infile mysvCsr1.csr -outfile mysvCsr1.cer`

### openssl使用

创建根私钥  
`openssl genrsa -out rootca.pem 2048`

生成自签证书  
`openssl req -x509 -new -key rootca.pem -out root.crt`

根据私钥生成公钥  
`openssl rsa -in rootca.pem -pubout -out rootcapub.pem`

生成客户端私钥  
`openssl genrsa -out clientkey.pem 2048`

生成客户端私钥证书申请文件  
`openssl req -new -key clientkey.pem -out client.csr`

使用根私钥签名客户端的申请文件  
`openssl x509 -req -in client.csr -CA root.crt -CAkey rootca.pem -CAcreateserial -days 3650 -out client.crt`

生成密钥库  
`openssl pkcs12 -export -in client.crt -inkey clientkey.pem -out client.p12`

以上命令使用了测试名称和默认参数，并在同一目录下生成文件，后续使用需要注意调整。

## 参考资料

[Android数字签名机制](https://duanqz.github.io/2017-09-01-Android-Digital-Signature#13-%E4%BD%BF%E7%94%A8)

[PCKS标准](https://www.iteye.com/blog/falchion-1472453)

[keystore工具](http://keystore-explorer.org/)

[SSL协议原理](https://blog.csdn.net/qq_38265137/article/details/90112705)

[netty使用ssl](https://blog.csdn.net/zhixinhuacom/article/details/79126274)