KeepAlived 的作用, 保证lvs 的高可用



架构模型: 

node1, node4: lvs + keepalived 高可用

node2, node3: 具体业务服务器



在lvs 实验机器之上, 先清理node1 的vip和lvs 的负载记录:

```shell
# 清空lvs 的负载
> ipvsadm -C

# 删掉vip 的配置
> ifconfig enp0s3:8 down
```



node2, node3 保持lvs 的实验的配置不动



node1, node4 上安装keepAlived: 

```shell
# 理论上keepalived 可以替换ipvsadm, ipvsadm 可以不装
> yum install -y keepalived ipvsadm

# 配置   /etc/keepalived/keepalived.conf
> cd /etc/keepalived/
# 备份配置文件
> cp keepalived.conf  keepalived.conf.bak
# vrrp: 虚拟路由
> vim keepalived.conf
```

node1 修改的keepalived.conf 的内容: 

```conf
vrrp_instance VI_1 {
    state MASTER			# node4 修改为 BACKUP
    interface enp0s3	# 根据具体机器的网卡调整
    virtual_router_id 51		
    priority 100			# node4 需改为 50
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass 1111
    }
    virtual_ipaddress {
    		# 相当于vip 的配置
        192.168.1.100/24 dev enp0s3 label enp0s3:8		# 需要改动
    }
}

# 这个ip 相当于ipvsadm -A 的操作ip和端口
virtual_server 192.168.1.100 80 {
    delay_loop 6
    # 轮询
    lb_algo rr
    # 模式 NAT, DR, TUN
    lb_kind DR
    # keepalived 记录请求的资源超时时间
    persistence_timeout 0
    protocol TCP
		
		# rip 的配置
    real_server 192.168.1.11 80 {
        weight 1
        # 检查real 服务的健康状态
        HTTP_GET {
        		# 检查的url 地址, 可多个
            url {
              path /
              status_code 200
            }
            
            connect_timeout 3
            nb_get_retry 3
            delay_before_retry 3
        }
    }
    
    # 第二个real 服务配置
    real_server 192.168.1.12 80 {
        weight 1
        HTTP_GET {
            url {
              path /
              status_code 200
            }
            connect_timeout 3
            nb_get_retry 3
            delay_before_retry 3
        }
    }
}
```

node1 拷贝keepalived.conf 配置文件到node4: 

```shell
> scp keepalived.conf root@192.168.1.13:`pwd`
```



node1 启动keepalived: 

```shell
> service keepalived start
```

启动服务后查看ifconfig 发现vip 已经新建好, 然后访问vip[192.168.1.100] 不成功.

再次修改keepalived.conf 文件: 

```
global_defs {
   notification_email {
     acassen@firewall.loc
     failover@firewall.loc
     sysadmin@firewall.loc
   }
   notification_email_from Alexandre.Cassen@firewall.loc
   smtp_server 192.168.200.1
   smtp_connect_timeout 30
   router_id LVS_DEVEL
   vrrp_skip_check_adv_addr
   # 删除 vrrp_strict 设置重启keepalived 访问成功
   vrrp_strict
   vrrp_garp_interval 0
   vrrp_gna_interval 0
}
```



keepalived 主备都启动的时候发现 备机[node4] `ifconfig` 也有vip 信息.

处理方法: 

```
# 查看selinux 信息
> getenforce
# 结果: 
# Enforcing         #强制开启
# Permissive        #宽容模式
# Disabled          #关闭
> setenforce 0  		# 设置为宽容模式
```

然后重启keepalived, 发现没有了vip信息.

参考: `http://www.capjsj.cn/keepalived_vip.html`



问题: 

当keepalived 变得不可靠的时候, 比如主机[异常推出]上 `kill -9 <进程id>`

会发现keepalived 主备都有vip信息, 会导致请求包紊乱.



处理方法: 

通过zk 保证keepalived 的可靠性