linux 环境安装了docker, 配置了一个案例的源文件, 导致 yum install -y net-tools 失败



查看当前源文件列表: 

> yum repolist all

开启关闭: 

> yum-config-manager --disable kfp63jaj.mirror.aliyuncs.com
>
> yum-config-manager --enable kfp63jaj.mirror.aliyuncs.com



查看本地的源文件路径: 

> /etc/yum.repos.d



**线上问题排查: **

```shell
# 查看当前系统使用情况, 进程
> top

# 查找cpu 使用率较高
> top -H -p pid
# 得到16进制的 tid
> printf '%x\n' pid
# 查找响应的堆栈信息, 关注waiting, timed_waiting, blocked
> jstack pid|grep 'tid' -C5 -color

# gc 情况
> jstat -gc pid 1000

#上下文切换
> vmstat 1

# 磁盘
> df -hl
> iostat -d -k -x
```



