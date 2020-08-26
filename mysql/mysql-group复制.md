**my.cnf配置文件调整: **

mgr-master节点的my.cnf文件: 

```
# Replication Framework
server_id=1						#修改
gtid_mode=ON
enforce_gtid_consistency=ON
master_info_repository=TABLE
relay_log_info_repository=TABLE
binlog_checksum=NONE
log_slave_updates=ON
log_bin=binlog
binlog_format=ROW

plugin_load_add='group_replication.so'
transaction_write_set_extraction=XXHASH64
group_replication_group_name="aaaaaaaa-aaaa-aaaa-aaaa-aaaaaaaaaaaa"
group_replication_start_on_boot=off
group_replication_local_address= "mgr-master:33061"		#修改
group_replication_group_seeds= "mgr-master:33061,mgr-slave-1:33061,mgr-slave-2:33061"
group_replication_bootstrap_group=off
```

mgr-slave-1的my.cnf配置文件: 

```cnf
# Replication Framework
server_id=11				#修改
gtid_mode=ON
enforce_gtid_consistency=ON
master_info_repository=TABLE
relay_log_info_repository=TABLE
binlog_checksum=NONE
log_slave_updates=ON
log_bin=binlog
binlog_format=ROW

plugin_load_add='group_replication.so'
transaction_write_set_extraction=XXHASH64
group_replication_group_name="aaaaaaaa-aaaa-aaaa-aaaa-aaaaaaaaaaaa"
group_replication_start_on_boot=off
group_replication_local_address= "mgr-slave-1:33061"		#修改
group_replication_group_seeds= "mgr-master:33061,mgr-slave-1:33061,mgr-slave-2:33061"
group_replication_bootstrap_group=off
```

mgr-slave-2的my.cnf配置文件: 

```
# Replication Framework
server_id=12			#修改
gtid_mode=ON
enforce_gtid_consistency=ON
master_info_repository=TABLE
relay_log_info_repository=TABLE
binlog_checksum=NONE
log_slave_updates=ON
log_bin=binlog
binlog_format=ROW

plugin_load_add='group_replication.so'
transaction_write_set_extraction=XXHASH64
group_replication_group_name="aaaaaaaa-aaaa-aaaa-aaaa-aaaaaaaaaaaa"
group_replication_start_on_boot=off
group_replication_local_address= "mgr-slave-2:33061"		#修改
group_replication_group_seeds= "mgr-master:33061,mgr-slave-1:33061,mgr-slave-2:33061"
group_replication_bootstrap_group=off
```

配置完成之后重启mysql服务



mgr-master在mysql-client设置: 

**设置复制账号权限: **

```mysql
SET SQL_LOG_BIN=0;
CREATE USER rpl_user@'%' IDENTIFIED BY 'rpl_user';
GRANT REPLICATION SLAVE ON *.* TO rpl_user@'%';
FLUSH PRIVILEGES;
SET SQL_LOG_BIN=1;
```

**指定恢复渠道channel:**

```mysql
CHANGE MASTER TO MASTER_USER='rpl_user', MASTER_PASSWORD='rpl_user' FOR CHANNEL 'group_replication_recovery';
```

**查看安装的plugin: **

```mysql
> show plugins;
```

**开启组复制: **

```mysql
# 设置group_replication_bootstrap_group为ON是为了标示以后加入集群的服务器以这台服务器为基准，以后加入的就不需要设置
SET GLOBAL group_replication_bootstrap_group=ON;
START GROUP_REPLICATION;
SET GLOBAL group_replication_bootstrap_group=OFF;
```

**查看mgr的状态: **

```mysql
# 查询表performance_schema.replication_group_members
select * from performance_schema.replication_group_members;
```



mgr-slave[1-2相同配置] 在mysql-client设置: 

**设置复制账号权限: **

```mysql
SET SQL_LOG_BIN=0;
CREATE USER rpl_user@'%' IDENTIFIED BY 'rpl_user';
GRANT REPLICATION SLAVE ON *.* TO rpl_user@'%';
FLUSH PRIVILEGES;
SET SQL_LOG_BIN=1;
```

**指定恢复渠道channel: **

```mysql
CHANGE MASTER TO MASTER_USER='rpl_user', MASTER_PASSWORD='rpl_user' FOR CHANNEL 'group_replication_recovery';
```

**查看安装的plugins: **

```mysql
show plugins;
```

**开启组复制: **

```mysql
# 这里不再需要开启group_replication_bootstrap_group，由于复制组已经被创建了，只需要将第二个节点添加进去即可
START GROUP_REPLICATION;
```

**查看mgr的状态: **

```mysql
# 查询表performance_schema.replication_group_members
select * from performance_schema.replication_group_members;
```



**查看当前mgr中的主节点: **

```mysql
# 返回主节点的server_uuid
SHOW STATUS LIKE 'group_replication_primary_member'
```



#### 自动选取主节点[master]

**停掉原始master节点: **

```mysql
# 停掉master节点的组复制。会重新选去一个master节点
stop group_replication;
# 开启当前节点的组复制, 成为slave 节点
start group_replication;
```

**查看相关状态: **

```mysql
# 当前组成员列表
select * from performance_schema.replication_group_members;
# 当前节点详细日志应用信息
select * from performance_schema.replication_group_member_stats；
# 当前复制渠道连接信息
select * from performance_schema.replication_connection_status;
# 当前复制渠道应用信息
select * from performance_schema.replication_applier_status;
# 当前主master
select a.variable_value,b.member_host,b.member_port,member_state from performance_schema.global_status a ,performance_schema.replication_group_members b where a.variable_value=b.member_id and variable_name= 'group_replication_primary_member';
```



#### mgr动态新增节点

##### 新增mgr-slave-3

**修改my.cnf: **

```mysql
# ....
server_id=13

# ....
group_replication_local_address= "mgr-slave-3:33061"		#修改
group_replication_group_seeds= "mgr-master:33061,mgr-slave-1:33061,mgr-slave-2:33061,mgr-slave-3:33061"
```

**设置复制账号权限: **

```mysql
SET SQL_LOG_BIN=0;
CREATE USER rpl_user@'%' IDENTIFIED BY 'rpl_user';
GRANT REPLICATION SLAVE ON *.* TO rpl_user@'%';
FLUSH PRIVILEGES;
SET SQL_LOG_BIN=1;
```

**指定恢复渠道channel: **

```mysql
CHANGE MASTER TO MASTER_USER='rpl_user', MASTER_PASSWORD='rpl_user' FOR CHANNEL 'group_replication_recovery';
```

**查看安装的插件plugins: **

```mysql
show plugins;
```

**集群中已存在的节点配置修改group_replication_group_seeds: **

```mysql
# 分别在mgr-master, mgr-slave-1, mgr-slave-2 中执行
set global  group_replication_group_seeds="mgr-master:33061,mgr-slave-1:33061,mgr-slave-2:33061,mgr-slave-3:33061";

# 将对应的值写入到my.cnf 中持久化
```

**开启组复制: **

```mysql
# 这里不再需要开启group_replication_bootstrap_group，由于复制组已经被创建了，只需要将新增节点添加进去即可
START GROUP_REPLICATION;
```

**查看mgr状态: **

```mysql
# 查询表performance_schema.replication_group_members
select * from performance_schema.replication_group_members;
```



#### mgr动态删除节点

**停止mgr-slave-3上的组复制: **

```mysql
# slave关闭后就被移除了组成员
STOP GROUP_REPLICATION;
```



**集群中剩下的节点配置修改group_replication_group_seeds: **

```mysql
set global  group_replication_group_seeds="mgr-master:33061,mgr-slave-1:33061,mgr-slave-2:33061";

# 将对应的值写入到my.cnf 中持久化
```

**彻底清理mgr-slave-3节点的相关组信息: **

```mysql
set global  group_replication_group_seeds="";
set global group_replication_local_address="";

# 修改my.cnf 进行持久化
```

**删除插件plugin, 复制账号**



#### proxysql实现mgr读写分离

参考: 

`https://www.cnblogs.com/kevingrace/p/10384691.html`

`https://blog.csdn.net/d6619309/article/details/54602556`



缺点: 

- 引入中间件proxysql, 增加及其资源的同时增加了运维的难度和复杂度
- proxy存在单点, 要做到高可用, 需要考虑更多[HAProxy, KeepAlived]