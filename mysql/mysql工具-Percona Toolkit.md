#### Persona Toolkit安装

centos: 

```shell
> apt-get install percona-toolkit
```



#### pt-heartbeat的使用

作用: 检测主从库的延迟情况.

**参数解析: **

```
// 后台执行
--daemonize

// 连接数据库的账号
--user, -u 

// 连接数据库的名称
--database, -D

// 连接数据库的地址
--host, -h

// 连接数据库的密码
--password, -p

// 连接数据库的端口
--port, -P

// [--frames=1m,2m,3m], 可以只指定一个.多个用逗号隔开。可用单位有秒（s）、分钟（m）、小时（h）、天（d）.
--frames
```



在master 节点运行: 

```
> pt-heartbeat -D test --update --user root --password root -h mysql-master --create-table --daemonize
```



在slave 节点运行: 

**一直执行, 连续打印: **

```
> pt-heartbeat -D test --monitor --user root --password root -h mysql-slave-1
```

**运行结果: **

```
// 表示1m, 5m, 15m延迟[可在--frames中设置]
0.00s [  0.00s,  0.00s,  0.00s ] 
0.00s [  0.00s,  0.00s,  0.00s ]
0.00s [  0.00s,  0.00s,  0.00s ]
0.00s [  0.00s,  0.00s,  0.00s ]
0.00s [  0.00s,  0.00s,  0.00s ]
```

**检查一次, 运行完退出: **

```
> pt-heartbeat -D test --check --user root --password root  h=mysql-slave-1
```

**运行结果: **

```
0.00
```



#### pt-table-checksum的使用

作用: 检测主从库是否一致

检测`test.heartbeat`表的是否一致: 

```
pt-table-checksum --nocheck-replication-filters --replicate=monitor.checksums --databases=test --tables=heartbeat h=mysql-slave-1,u=root,p=root,P=3306
```

会出现如下提示信息: 

> Checking if all tables can be checksummed ...
> Starting checksum ...
> Replica 16a42100ba00 has binlog_format ROW which could cause pt-table-checksum to break replication.  Please read "Replicas using row-based replication" in the LIMITATIONS section of the tool's documentation.  If you understand the risks, specify --no-check-binlog-format to disable this check.



所以添加`--no-check-binlog-format`: 

```
pt-table-checksum --no-check-binlog-format --replicate=monitor.checksums --databases=test --tables=heartbeat h=mysql-slave-1,u=root,p=root,P=3306
```

> Checking if all tables can be checksummed ...
> Starting checksum ...
> Diffs cannot be detected because no slaves were found.  Please read the --recursion-method documentation for information.
>             TS ERRORS  DIFFS     ROWS  DIFF_ROWS  CHUNKS SKIPPED    TIME TABLE
> 05-10T03:23:37      0      0        1          0       1       0   0.018 test.heartbeat





#### pt-table-sync的使用

作用: 主备数据同步



**同步所有的库表: **

```
pt-table-sync --charset=utf8--ignore-databases=mysql,sys,percona dsn=u=root,p=root,h=mysql-master,P=3306 dsn=u=root,p=root,h=mysql-slave-1,P=3306 --execute --print
```



**同步指定库指定表: **

```
pt-table-sync --charset=utf8--ignore-databases=mysql,sys,percona --databases=test1 --no-check-slave dsn=u=root,p=root,h=mysql-master,P=3306 dsn=u=root,p=root,h=mysql-slave-1,P=3306 --execute --print
```



指定表: 

```
pt-table-sync --charset=utf8--ignore-databases=mysql,sys,percona --databases=test5 --tables=test_nu --no-check-slave dsn=u=root,p=root,h=mysql-master,P=3306 dsn=u=root,p=root,h=mysql-slave-1,P=3306 --execute --print
```

同步指定表也可以: 

```
pt-table-sync --charset=utf8--ignore-databases=mysql,sys,percona --tables=test5.test_nu  --no-check-slave dsn=u=root,p=root,h=mysql-master,P=3306 dsn=u=root,p=root,h=mysql-slave-1,P=3306 --execute --print
```





