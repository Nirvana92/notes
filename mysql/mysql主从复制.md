### mysql主从复制

mysql 复制同步类型:

- 单向异步复制[原始类型]

```
其中一台服务器充当主服务器, 而一台或多台其他服务器充当从服务器.这与 NDB Cluster 同步复制相反.
```

参考: `https://dev.mysql.com/doc/refman/5.7/en/mysql-cluster.html`

![image-20200510001933514](/Users/mac/Documents/笔记/mysql/image-20200510001933514.png)

- 半同步复制-通过安装插件实现

```
在返回到执行事务的会话之前, 在主块上执行的提交, 直到至少一个从属确认它已接受并记录了事务的事件为止.
```

参考: `https://dev.mysql.com/doc/refman/5.7/en/replication-semisync.html`

安装配置参考: `https://dev.mysql.com/doc/refman/5.7/en/replication-semisync-installation.html`

![image-20200510002017314](/Users/mac/Documents/笔记/mysql/image-20200510002017314.png)

- 延迟复制

```
以便从属服务故意落后于住服务器至少指定的时间量. 默认延迟0s, 使用MASTER_DELAY进行设置
```

设置: 

```
> CHANGE MASTER TO MASTER_DELAY = N;
> STOP SLAVE;
> START SLAVE;

// 将延迟重置为0
> RESET SLAVE;
```

`show slave status: 有三个字段可提供有关延迟的信息: `

1. sql_delay: 非负整数, 指示从站必须落后主站的秒数
2. sql_remaining_delay: 
3. slave_sql_running_state: 

参考: `https://dev.mysql.com/doc/refman/5.7/en/replication-delayed.html`



**复制格式的类型:**

- 基于语句的复制[SBR(`Statement-based Replication`)]-复制整个sql语句[5.7.7以前默认]

- 基于行的复制[RBR(`row-based Replication`)]-仅复制更改的行[>=5.7.7默认]

- 混合复制[MBR(`mixed-based Replication`)]

更多信息可参考: `https://dev.mysql.com/doc/refman/5.7/en/replication-formats.html`

#### 设置基于二进制日志文件位置的复制

1. **主服务器上, 启动二进制日志记录并配置唯一的服务器id.重启mysql**
2. **每个从服务器, 必须配置唯一的服务器id, 重启mysql**
3. **[可选] 创建单独的用户, 供从属使用, 以便在读取二进制日志进行复制时与主控一起使用**
4. **获取复制主节点二进制日志坐标**



- **主服务器上, 启动二进制日志记录并配置唯一的服务器id.重启mysql**

```
// 编辑mysql 的配置文件
> vim /etc/mysql/mysql.conf.d/mysqld.cnf

// 添加如下配置:
log-bin=mysql-bin
server-id=1
```

**note:**

1. 为了在InnoDB 与事务一起使用的复制设置中获得最大的持久性和一致性, 在配置文件中添加如下配置

```
innodb_flush_log_at_trx_commit=1
sync_binlog=1
```

**innodb_flush_log_at_trx_commit 系统参数:**

控制提交操作的严格遵从acid, 参考: `https://dev.mysql.com/doc/refman/5.7/en/innodb-parameters.html#sysvar_innodb_flush_log_at_trx_commit`

**可选值:**

**0:**每秒写入一次日志并将其刷新到磁盘, 尚未刷新日志的事务可能会在崩溃中丢失

**1:**完全符合acid[默认值]

**2:** 每次事务提交后写入日志, 并每秒刷新一次到磁盘, 尚未刷新日志的事务可能在崩溃中丢失

**sync_binlog 系统参数: **

控制mysql服务器将二进制日志同步到磁盘的频率, 参考: `https://dev.mysql.com/doc/refman/5.7/en/replication-options-binary-log.html#sysvar_sync_binlog`

取值范围: 

`1: 默认值`

`0: 最低值`

`4294967295:最大值`

`sync_binlog=0:禁用mysql服务器将二进制日志同步到磁盘功能.取而代之的, mysql服务器依靠操作系统不时的将二进制日志刷新到磁盘上。此设置提供最佳性能, 但在电源故障或操作系统崩溃的情况下, 服务器可能提交了尚未同步到二进制日志的事务`

`sync_binlog=1:最安全的设置, 提交事务之前启用二进制日志到磁盘同步,如果发生电源故障或系统崩溃, 二进制日志中缺少的事务将仅处于准备状态`

`sync_binlog=N:n是二进制日志提交组已后记之后, 二进制日志将同步到磁盘`

为了在InnoDB与事务一起使用的复制设置中获得最大的持久性和一致性, 可以设置: 

```
innodb_flush_log_at_trx_commit=1
sync_binlog=1
```



2. 确保 skip_networking 变量未被启动

`--skip_networking[={OFF|ON}]`

此变量控制服务器是否允许tcp/ip 连接, 默认情况下, 它是禁用的[允许tcp连接], 如果启用, 则服务器仅允许本地[非tcp/ip]连接

- **每个从服务器, 必须配置唯一的服务器id, 重启mysql**

配置唯一服务id: 

```
> vim /etc/mysql/mysql.conf.d/mysqld.cnf

// 添加如下配置: 
server-id=21
read-only=1	// 设置只读
```



配置信息: 

```
> change master to 
						master_host='mysql-master', 
						master_user='root', 
						master_password='root', 
						// 这个值设置master 启动后的bin-log 的名字, 如:mysql-bin.000001
						master_log_file='mysql-bin';
            
            // 这个配置的时候走的默认值
						# master_log_pos='';
						
> start slave;
```

**查看主从是否配置成功: **

```
> show slave status \G;

// 看到如下信息, 可知配置成功生效
Slave_IO_Running: Yes
Slave_SQL_Running: Yes
```

注意: 

当时配置的时候, 出现 `Slave_IO_Running: No`, 后来手动设置`stop slave; change master to set Master_Log_File='mysql-bin.000001';   start slave;` 在查看发现IO_Running: Yes, 主从已经配置成功。

- **[可选] 创建单独的用户, 供从属使用, 以便在读取二进制日志进行复制时与主控一起使用**

- **获取复制主节点二进制日志坐标**

```
// 查看master 节点二进制文件信息
> show master status;
```



#### 复制原理

不同复制方式的优缺点参考: `https://dev.mysql.com/doc/refman/5.7/en/replication-sbr-rbr.html`

1. 基于语句的复制[SBR]-复制整个sql语句

```
优点: 
1. 实现比较简单, 记录和执行这些语句, 就可以让主备保持同步.
2. 不需要占用太多的带宽. 一条更新好几兆数据的语句在二进制日志可能只占用几十个字节.

缺点: 
1. 主从执行的时间会有出入, 一些元数据信息, 如当前时间戳。会不同.
2. 更新必须是串行的.这需要更多的锁.
```



2. 基于行的复制[RBR]-仅复制更改的行[基于bin-log]

```
恶终方式会讲实际数据记录在二进制日志中, 最大的好处就是可以正确的复制每一行的数据, 一些语句可以被更加有效的复制

优点: 
1. 比如多表联合操作, 大表汇总成小表.通过行复制, 最终可能只有一两条新增数据。

缺点: 
1. 如果更新全表update 语句, 基于行的复制, 可能代价更大。
```

3. 混合复制[MBR]

备注: 可以通过 `binlog_format` 变量设置模式.[STATEMENT, MIXED, ROW]



#### 测试备库延迟

最好的解决办法是 `heartbeat record`, 包含在`Percona Toolkit` 中的`pt-heartbeat` 脚本是'复制心跳'最流行的一种实现.



#### 确定主备是否一致

问题描述: 备库可能因为mysql自身的特性导致数据不一致, 例如mysql的bug, 网络中断, 服务器崩溃, 非正常关闭或者其他一些错误.

`Persona Toolkit` 中的``pt-table-checksum` 能够解决问题.



#### 从主库重新同步备库

`Persona Toolkit` 中的`pt-table-sync` 可以很好的解决该问题



#### 配置/监控/管理和优化工具

`Percona Toolkit`

`Percona XtraBackup`









