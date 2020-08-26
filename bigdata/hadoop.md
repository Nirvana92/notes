![hadoop-ha](/Users/mac/Documents/笔记/bigdata/hadoop-ha.png)

机器角色分配: 

|       | NN   | DN   | JN   | ZKFC | ZK   |
| ----- | ---- | ---- | ---- | ---- | ---- |
| node1 | 1    |      |      | 1    |      |
| node2 | 1    | 1    | 1    | 1    | 1    |
| node3 |      | 1    | 1    |      | 1    |
| node4 |      | 1    | 1    |      | 1    |

hadoop 高可用[HA]配置: 

`hdfs-site.xml`

```xml
<!-- 新名称服务的逻辑名称 -->
<property>
  <name>dfs.nameservices</name>
  <value>mycluster</value>
</property>
<!-- 服务中每个NameNode的唯一标识符. 2台, 最多不超过5个 -->
<property>
  <name>dfs.ha.namenodes.mycluster</name>
  <value>node1,node2</value>
</property>
<!-- 每个NameNode监听的标准RPC地址 -->
<property>
  <name>dfs.namenode.rpc-address.mycluster.node1</name>
  <value>node1:8020</value>
</property>
<property>
  <name>dfs.namenode.rpc-address.mycluster.node2</name>
  <value>node2:8020</value>
</property>
<!-- 每个NameNode监听的标准HTTP地址 -->
<property>
  <name>dfs.namenode.http-address.mycluster.node1</name>
  <value>node1:50070</value>
</property>
<property>
  <name>dfs.namenode.http-address.mycluster.node2</name>
  <value>node2:50070</value>
</property>
<!-- 标识NameNode将在其中写入/读取编辑内容的JN组的URI -->
<property>
  <name>dfs.namenode.shared.edits.dir</name>
  <value>qjournal://node1:8485;node2:8485/mycluster</value>
</property>
<!-- JournalNode守护程序将存储其本地状态的路径 -->
<property>
  <name>dfs.journalnode.edits.dir</name>
  <value>/var/bigdata/hadoop/ha/dfs/jn</value>
</property>
<!-- HDFS客户端用于联系活动NameNode的Java类 -->
<property>
  <name>dfs.client.failover.proxy.provider.mycluster</name>
  <value>org.apache.hadoop.hdfs.server.namenode.ha.ConfiguredFailoverProxyProvider</value>
</property>

<!-- SSH到Active NameNode并终止进程 -->
<property>
  <name>dfs.ha.fencing.methods</name>
  <value>sshfence</value>
</property>
<property>
  <name>dfs.ha.fencing.ssh.private-key-files</name>
  <value>/root/.ssh/id_rsa</value>
</property>
<!-- 自动故障转移的配置需要在配置中添加两个新参数 -->
<property>
  <name>dfs.ha.automatic-failover.enabled</name>
  <value>true</value>
</property>
```

`core-site.xml`

```xml
<!-- Hadoop FS客户端使用的默认路径前缀 -->
<property>
  <name>fs.defaultFS</name>
  <value>hdfs://mycluster</value>
</property>
<!-- 这指定应为自动故障转移设置群集 -->
<property>
  <name>ha.zookeeper.quorum</name>
 <value>node2:2181,node3:2181,node4:2181</value>
</property>
```



node2 生成公钥追加到node1 的authorized_keys 中

启动jn: 

```shell
$> hadoop-daemon.sh start journalnode
```

格式化 namenode[搭建第一次做如下操作, 以后均不用做次操作]: 

```shell
$> hdfs namenode -format
```

启动node1的namenode: 

```shell
$> hadoop-daemon.sh start namenode
```

node2的namenode同步: 

```shell
$> hdfs namenode -bootstrapStandby
```

格式化zk: 

```shell
$> hdfs zkfc -formatZK
```

启动cluster: 

```shell
$> start-dfs.sh
```

手动启动cluster: 

```shell
$> hdfs --daemon start zkfc
```





hdfs 权限: 默认使用操作系统提供的的用户

扩展 LDAP





配置yarn:

`mapred-site.xml:`

```xml
<property>
  <name>mapreduce.framework.name</name>
  <value>yarn</value>
</property>
```

`yarn-site.xml:`

```xml
<property>
  <name>yarn.nodemanager.aux-services</name>
  <value>mapreduce_shuffle</value>
</property>
<property>
  <name>yarn.resourcemanager.ha.enabled</name>
  <value>true</value>
</property>
<property>
  <name>yarn.resourcemanager.cluster-id</name>
  <value>cluster1</value>
</property>
<property>
  <name>yarn.resourcemanager.ha.rm-ids</name>
  <value>rm1,rm2</value>
</property>
<property>
  <name>yarn.resourcemanager.hostname.rm1</name>
  <value>node3</value>
</property>
<property>
  <name>yarn.resourcemanager.hostname.rm2</name>
  <value>node4</value>
</property>
<property>
  <name>yarn.resourcemanager.webapp.address.rm1</name>
  <value>node3:8088</value>
</property>
<property>
  <name>yarn.resourcemanager.webapp.address.rm2</name>
  <value>node4:8088</value>
</property>
<property>
  <name>hadoop.zk.address</name>
  <value>node2:2181,node3:2181,node4:2181</value>
</property>
```

