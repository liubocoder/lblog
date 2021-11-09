---
layout: post
title:  "搭建VPN服务器"
categories: 后端 网络
tags:  openvpn docker
author: liubo
typora-root-url: ..
---

* content
{:toc}


## 需求
有时候需要查阅资料，不得不借助各种梯子，都不是很稳定，风险不能自己掌控，很难受。。于是想用openvpn自己搭建一个。

## 准备工具
    1. 云服务器，可以买阿里云或者腾讯云，注意是否有跨境需求决定服务器位置
    2. 云服务器上安装docker和openvpn
    3. 客户端linux使用openvpn，windows使用OpenVpnGUI，android上使用OpenVpnAPP




## 使用脚本配置服务器
便于后续服务器迁移，使用脚本进行配置
```
#!/bin/sh
#配置文件存放目录
DIR="/root/openvpn/ovpn-data"
#docker 容器名称
NAME="openvpn"
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
            docker run -v ${DIR}:/etc/openvpn --rm kylemanna/openvpn ovpn_genconfig -u ${PROTO}://${DSTIP}
            docker run -v ${DIR}:/etc/openvpn --rm -it kylemanna/openvpn ovpn_initpki
            sed "s/port .*/port ${PORT}/" -i ${DIR}/openvpn.conf
            sed "s/declare -x OVPN_PORT=.*/declare -x OVPN_PORT=${PORT}/" -i ${DIR}/ovpn_env.sh
        return
        ;;
        "run")
            docker stop ${NAME} >/dev/null 2>&1
            docker rm ${NAME} >/dev/null 2>&1
            sed "s/port .*/port ${PORT}/" -i ${DIR}/openvpn.conf
            sed "s/declare -x OVPN_PORT=.*/declare -x OVPN_PORT=${PORT}/" -i ${DIR}/ovpn_env.sh
            docker run --name ${NAME} -v ${DIR}:/etc/openvpn -d --net host --cap-add=NET_ADMIN kylemanna/openvpn
        return
        ;;
        "product")
            docker run -v ${DIR}:/etc/openvpn --rm -it kylemanna/openvpn easyrsa build-client-full $2 nopass
            docker run -v ${DIR}:/etc/openvpn --rm kylemanna/openvpn ovpn_getclient $2> $2.ovpn
        return
        ;;
        "log")
                docker exec ${NAME} cat /tmp/openvpn-status.log
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
sudo openvpn --daemon --config xx.ovpn
```

2. windows导入配置即可

    ![img](/assets/blog-img/server-vpn-1-2.png)

3. android也直接导入配置

    ![img](/assets/blog-img/server-vpn-1-1.png)

## 参考资料

<https://github.com/kylemanna/docker-openvpn>

<https://github.com/OpenVPN/openvpn>
