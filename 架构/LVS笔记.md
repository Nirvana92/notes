> RFC1918规定了三个保留地址段:
>
> 不向特定的用户分配, 被IANA作为私有地址保留, 仅能在内部使用, 不能作为全球路由地址.
>
> A: 10.0.0.0 - 10.255.255.255
>
> B: 172.16.0.0 - 172.31.255.255
>
> C: 192.168.0.0 - 192.168.255.255



> NAT 模式流程图: 

> 处理客户端ip(CIP)在负载层包装相同的方式通过在路由器中对应表中修改为不同的端口(port)做映射, 保证ip对ip。

S-NAT模式: 

![image-20200512104258442](/Users/mac/Documents/笔记/架构/image-20200512104258442.png)

D-NAT模式: 

![image-20200512102537411](/Users/mac/Documents/笔记/架构/image-20200512102537411.png)





> DR 模型: 

> 解决NAT 模型的负载瓶颈问题, 请求在负载层直接修改的物理层(mac), 每个服务对外隐藏vip.

**隐藏vip方法: 对外隐藏, 对内可见**

```
kernel parameter

目标mac 地址为全F, 交换机出发广播

/proc/sys/net/ipv4/conf/*IF*/
arp_ignore: 定义接受到ARP请求时的响应级别: 
0: 只要本地配置的有相应地址, 就给予响应
1: 仅在请求的目标(mac)地址配置请求到达的接口上的时候, 才给予响应

arp_announce: 定义将自己地址向外通告时的通告级别: 
0: 将本地任何接口上的任何地址向外通告
1: 试图仅向目标网络通告与其网络匹配的地址
2: 仅向与本地接口上地址匹配的网络进行通告
```



![image-20200512110421385](/Users/mac/Documents/笔记/架构/image-20200512110421385.png)

> TUN 隧道技术

> CIP -> VIP 外层包裹了一层DIP -> RIP

![image-20200512150554868](/Users/mac/Documents/笔记/架构/image-20200512150554868.png)

> LVS DR模型配置

![image-20200512151841642](/Users/mac/Documents/笔记/架构/image-20200512151841642.png)

![image-20200512151858313](/Users/mac/Documents/笔记/架构/image-20200512151858313.png)

![image-20200512154353240](/Users/mac/Documents/笔记/架构/image-20200512154353240.png)

**操作: **

node1上添加子网络, 192.168.1.100

```shell
> ifconfig enp0s3:8 192.168.1.100/24
```

调整node2, node3 调整arp协议并添加隐藏vip: 

```shell
> cd /proc/sys/net/ipv4/conf/enp0s3/
> echo 1 > arp_ignore
> echo 2 > arp_announce

# 去到all 目录下修改arp_ignore, arp_announce
> cd /proc/sys/net/ipv4/conf/all
> echo 1 > arp_ignore
> echo 2 > arp_announce

# 配置隐藏vip. 
> ifconfig lo:2 192.168.1.100 netmask 255.255.255.255
```

在node2, node3 上安装https并启动: 

```
> yum install -y httpd
> service httpd start
```

在node2, node3 上创建一个主页: 

```shell
# /var/www/html/ httpd的默认文件目录
> vim /var/www/html/index.html
# 在里面添加 from 192.168.1.1x
```

然后访问: 192.168.1.11 | 192.168.1.12 即可看到静态页面内容

注意: 如果现实无法访问, 可能是11 | 12 防火墙打开了, 关闭防火墙即可`systemctl stop firewalld.service`



在node1 上安装 ipvsadm

```shell
> yum install -y ipvsadm
```

node1 上添加进包规则: 

```shell
> ipvsadm -A -t 192.168.1.100:80 -s rr

> ipvsadm -a -t 192.168.1.100:80 -r 192.168.1.11 -g -w 1
> ipvsadm -a -t 192.168.1.100:80 -r 192.168.1.12 -g -w 1

# 查看
> ipvsadm -ln
```

验证: 访问 `192.168.1.100:80`, 即可看到负载

node2, node3 上验证: 

```shell
# 看到很多socket 连接
> netstat -natp 
```



查看统计信息[查看偷窥记录]: 

```shell
> ipvsadm -lnc
```



