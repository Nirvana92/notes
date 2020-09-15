查看 `redis`的信息: 

```shell
127.0.0.1:6379>  info [selection]
127.0.0.1:6379>  info server
```

输出内容: 

```shell
# Server
redis_version:6.0.6
redis_git_sha1:00000000
redis_git_dirty:0
redis_build_id:19d4277f1e8a2fed
redis_mode:standalone
os:Linux 4.19.76-linuxkit x86_64
arch_bits:64
multiplexing_api:epoll
atomicvar_api:atomic-builtin
gcc_version:8.3.0
process_id:1
run_id:da5a1dfd8f62fb31d7fdda6580c9575f74cf5edb
tcp_port:6379
uptime_in_seconds:1068
uptime_in_days:0
hz:10
configured_hz:10
lru_clock:6324002
executable:/data/redis-server
config_file:
```

查看 `redis` 日志级别: 

```shell
> 127.0.0.1:6379> config get loglevel
```

日志级别输出: 

```shell
1) "loglevel"
2) "notice"
```

获取所有的配置项: 

```shell
127.0.0.1:6379> config get *
```

编辑配置的方式: 

```shell
127.0.0.1:6379> config set [config_setting_name] [new_config_val]
```

比如: 

```shell
127.0.0.1:6379> config set loglevel "notice"
```



`redis` 相关的配置项参考: `runoob.com/redis/redis-conf.html`