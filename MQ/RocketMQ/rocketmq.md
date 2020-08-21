rocketmq 环境搭建: 在docker 中搭建

修改docker 的镜像地址: `https://6kx4zyno.mirror.aliyuncs.com`

参考博客: `https://www.cnblogs.com/kiwifly/p/11546008.html`

下载`rocketmq镜像`: 

**安装Namesrv: **

```shell
> docker pull rocketmqinc/rocketmq:4.4.0

# 启动容器
# 替换{RmHome} 为电脑需要保存MQ的日志与数据的地方
> docker run -d -p 9876:9876 -v {RmHome}/data/namesrv/logs:/root/logs -v {RmHome}/data/namesrv/store:/root/store --name rmqnamesrv -e "MAX_POSSIBLE_HEAP=100000000" rocketmqinc/rocketmq:4.4.0 sh mqnamesrv
```

**安装Broker 服务器:** 

1. 拉取镜像: 上面已经拉取过了就不需要。

2. 在 `{RmHome}/conf` 目录下创建 `boker.conf`文件

3. 在`broker.conf`中写入如下内容

   ```
   brokerClusterName = DefaultCluster
   brokerName = broker-a
   brokerId = 0
   deleteWhen = 04
   fileReservedTime = 48
   brokerRole = ASYNC_MASTER
   flushDiskType = ASYNC_FLUSH
   brokerIP1 = {本地外网 IP}
   ```

4. 启动容器

   ```shell
   docker run -d -p 10911:10911 -p 10909:10909 -v  {RmHome}/data/broker/logs:/root/logs -v  {RmHome}/rocketmq/data/broker/store:/root/store -v  {RmHome}/conf/broker.conf:/opt/rocketmq-4.4.0/conf/broker.conf --name rmqbroker --link rmqnamesrv:namesrv -e "NAMESRV_ADDR=namesrv:9876" -e "MAX_POSSIBLE_HEAP=200000000" rocketmqinc/rocketmq:4.4.0 sh mqbroker -c /opt/rocketmq-4.4.0/conf/broker.conf
   ```

   `{RmHome}` 需要替换成电脑配置的路径

5. 安装rocketmq 控制台

   拉取镜像

   ```shell
   > docker pull pangliang/rocketmq-console-ng
   ```

   启动容器: 

   ```shell
   docker run -e "JAVA_OPTS=-Drocketmq.namesrv.addr={本地外网 IP}:9876 -Dcom.rocketmq.sendMessageWithVIPChannel=false" -p 8080:8080 -t pangliang/rocketmq-console-ng
   ```

   