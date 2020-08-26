单机redis 存在的问题: 

- 单点故障
- 容量有限
- 压力



docker 启动redis: 

前提: 修改redis.conf 中的`daemonize yes` 为`daemonize no`

```
> docker run -p 6379:6379 --name redis -v /home/gzm/software/redis/redis.conf:/etc/redis/redis.conf -d redis:6.0.1
```



AKF: xyz x: 全量镜像 y: 按照业务需求拆分 z: 优先级, 逻辑再拆分



#### redis 复制

在从redis 中使用slaveof[REPLICAOF] 命令: 

```
方式1: 
在配置文件中加入 slaveof <masterHost> <masterPort>

方式2: 
redis-server 启动命令后加入 --slaveof <masterHost> <masterPort>

方式3: 
直接使用命令: slaveof <masterHost> <masterPort>
```



#### 哨兵[redis sentinel]

