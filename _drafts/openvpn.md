

npv服务架设遇到的问题


## opennpv服务器能收包，不能发包
梯子被禁了，运营商禁用了端口，无解。。只能换端口

## 连接npv不能访问外网
可能情况
1. net.ipv4.ip_forward=0

sysctl -a | grep forward 查询ipv4的转发情况，改为=1

2. Ubuntu上可能dns受限

在客户端的npv配置文件中追加如下配置
script-security 2
up /etc/opennpv/update-resolv-conf
down /etc/opennpv/update-resolv-conf

3. 查看本地的路由是否配置正确（一般不会出问题）









[open-npv原理](https://wenku.baidu.com/view/0b7bde886529647d27285277.html)