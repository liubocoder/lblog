---
layout: post
title:  "搭建NPV服务器"
categories: 后端 网络
tags:  opennpv docker
author: liubo
typora-root-url: ..
---

* content
{:toc}


## 需求
有时候需要查阅资料，不得不借助各种工具，都不是很稳定，风险不能自己掌控，很难受。。于是想用'opennpv'自己搭建一个，为了提交这篇文章，所有能改的都改为opennpv。

## 准备工具
    1. 云服务器，可以买阿里云或者腾讯云，注意是否有跨境需求决定服务器位置
    2. 云服务器上安装docker和opennpv
    3. 客户端linux使用opennpv，windows使用OpenNpvGUI，android上使用OpenNpvAPP




## 使用脚本配置服务器
便于后续服务器迁移，使用脚本进行配置
```
#!/bin/sh
#配置文件存放目录
DIR="/root/opennpv/ovpn-data"
#docker 容器名称
NAME="opennpv"
#端口
PORT=yourCustomPort
#服务器域名或IP
DSTIP=yourServerIp or domainName
#udp or tcp
PROTO=udp

function main {
    case $1 in 
        "setup")
            rm ${DIR} -rf
            mkdir -p ${DIR}
            docker run -v ${DIR}:/etc/opennpv --rm kylemanna/opennpv ovpn_genconfig -u ${PROTO}://${DSTIP}
            docker run -v ${DIR}:/etc/opennpv --rm -it kylemanna/opennpv ovpn_initpki
            sed "s/port .*/port ${PORT}/" -i ${DIR}/opennpv.conf
            sed "s/declare -x OVPN_PORT=.*/declare -x OVPN_PORT=${PORT}/" -i ${DIR}/ovpn_env.sh
        return
        ;;
        "run")
            docker stop ${NAME} >/dev/null 2>&1
            docker rm ${NAME} >/dev/null 2>&1
            sed "s/port .*/port ${PORT}/" -i ${DIR}/opennpv.conf
            sed "s/declare -x OVPN_PORT=.*/declare -x OVPN_PORT=${PORT}/" -i ${DIR}/ovpn_env.sh
            docker run --name ${NAME} -v ${DIR}:/etc/opennpv -d --net host --cap-add=NET_ADMIN kylemanna/opennpv
        return
        ;;
        "product")
            docker run -v ${DIR}:/etc/opennpv --rm -it kylemanna/opennpv easyrsa build-client-full $2 nopass
            docker run -v ${DIR}:/etc/opennpv --rm kylemanna/opennpv ovpn_getclient $2> $2.ovpn
        return
        ;;
        "log")
                docker exec ${NAME} cat /tmp/opennpv-status.log
                return
        ;;
    esac

    echo "err cmd $1"
}
```

直接将脚本保存到xx文件中
>- `./xx setup`创建vpn相关配置，这一步有很多需要输入名字或密码的地方，注意牢记密码，不要输错了
>- `./xx run` 运行服务器
>- `./xx product` 创建客户端密钥配置，注意需要输入密码
>- `./xx log` 查看服务器日志

## 客户端中使用

1. linux中使用命令
```
sudo opennpv --daemon --config xx.ovpn
```

2. windows导入配置即可

    ![img](/assets/blog-img/server-vpn-1-2.png)

3. android也直接导入配置

    ![img](/assets/blog-img/server-vpn-1-1.png)

## npv服务架设遇到的问题

### opennpv服务器能收包，不能发包
梯子被禁了，运营商禁用了端口，无解。。只能换端口

### 连接npv不能访问外网

可能情况
>- net.ipv4.ip_forward=0
```
sysctl -a | grep forward 查询ipv4的转发情况，改为=1
```
>- Ubuntu上可能dns受限
在客户端的npv配置文件中追加如下配置
```
script-security 2
up /etc/opennpv/update-resolv-conf
down /etc/opennpv/update-resolv-conf
```
>- 查看本地的路由是否配置正确（一般不会出问题）


## 参考资料

<https://github.com/kylemanna/docker-opennpv>

<https://github.com/OpenVPN/opennpv>
