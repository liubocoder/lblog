---
layout: post
title:  "Docker基础"
categories: 后端
tags:  docker
author: liubo
typora-root-url: ..
---

* content
{:toc}


## Docker常用命令
1. 仅创建镜像
`docker create`

2. 运行（不存在将创建镜像）
`docker run`
```
sudo docker run -d -it -p 8181:8181 --name onos gwsdn/onos:2.6.0
    -d deamon后台，--rm一次性容器，运行结束后删除
    -i 始终保持交互状态
    -t 终端
    -v 映射文件或目录，例如 -v /opt/conf/nginx.conf:/etc/xx.conf，注意宿主机地址不是‘/’或‘~/’开头，则是Volume机制
    -p 端口映射，默认是tcp端口  -p 80:8080/tcp，udp写为 -p 8181:8181/udp
    --net 映射网络 例如：host
```
[Volume机制参考这里](https://docs.docker.com/storage/volumes/)




3. 镜像备份 `docker save`
```
sudo docker save -o gwsdn-onos-2.6.0.tar gwsdn/onos:2.6.0
```

4. 镜像导入 `docker load`
```
sudo docker load -i gwsdn-onos-2.6.0.tar
    -i 从文件导入
```

5. 拷贝文件`docker cp`
```
sudo docker cp /masterdir/file 84b623993841:/slavedir/
```

6. 执行命令`docker exec`
```
sudo docker exec -it 84b623993841 /bin/bash
```

7. 批量执行命令  
首先需要过滤出要批量操作的ID，例如`docker ps | awk '{ print $1}' | tail -n +2`过滤所有运行容器ID  
停止所有正在运行的容器
```
docker stop $(docker ps  awk '{ print $1}' | tail -n +2)
```

## Docker图形界面管理器
我选用了Lazydocker，属于字符图形界面，可以鼠标或全键盘操作，最主要是可以在远程终端上使用，例如：SecureCRT。可按照如下步骤配置

```
wget https://github.com/jesseduffield/lazydocker/releases/download/v0.12/lazydocker_0.12_Linux_x86_64.tar.gz

tar zxf lazydocker_0.12_Linux_x86_64.tar.gz

sudo install lazydocker /usr/local/bin/
```

执行`lazydocker version`，出现如下界面，那么配置完成了

![img](/assets/blog-img/docker-base-1-1.png)

执行`sudo lazydocker`命令运行管理界面

![img](/assets/blog-img/docker-base-1-2.png)

## 参考资料

[Docker教程](https://www.runoob.com/docker/docker-tutorial.html)

[Docker官方文档](https://docs.docker.com/get-started/)

[Docker私服搭建](https://blog.csdn.net/lwcaicsdn/article/details/97775983)

[Lazydocker发布版本](https://github.com/jesseduffield/lazydocker/releases)

[Docker图形化管理工具](https://cloud.tencent.com/developer/article/1647274)