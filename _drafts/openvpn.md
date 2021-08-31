

vpn服务架设遇到的问题


## 连接vpn不能访问外网
可能情况
1. net.ipv4.ip_forward=0

sysctl -a | grep forward 查询ipv4的转发情况，改为=1