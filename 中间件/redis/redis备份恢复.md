 备份数据: 

```shell
127.0.0.1:6379> save
```

该命令将在 redis 安装目录中创建 `dump.rdb` 文件。

```shell
# 后台执行保存数据到 rdb
127.0.0.1:6379> BGSAVE
```

恢复数据: 

1. 查看redis 加载数据的目录

```shell
127.0.0.1:6379> config get dir
```

2. 将有数据的rdb 文件放到上面命令查询出来的路径中(dump.rdb)。

**注意: 如果通过执行 flushall 命令, 需要先备份一份rdb 文件, 不然保存的rdb 文件为空** 

禁用危险命令: 

```shell
# 编辑redis.conf 文件
rename-command flushall "xxxxxx"
rename-command keys "xxxxxx"
rename-command config "xxxxxx"
rename-command flushdb "xxxxxx"
```

