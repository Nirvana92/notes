##### docker 安装

下载`Elasticsearch`镜像: 

```shell
> docker pull elasticsearch:7.6.2
```

运行命令: 

```shell
> docker run -d -e "discovery.type=single-node" -p 9200:9200  elasticsearch:7.6.2
```

注意: 

直接运行: `docker run elasticsearch:7.6.2`会报如下错误: 

```
ERROR: [1] bootstrap checks failed
[1]: the default discovery settings are unsuitable for production use; at least one of [discovery.seed_hosts, discovery.seed_providers, cluster.initial_master_nodes] must be configured
ERROR: Elasticsearch did not exit normally - check the logs at /usr/share/elasticsearch/logs/docker-cluster.log
{"type": "server", "timestamp": "2020-05-03T10:12:51,335Z", "level": "INFO", "component": "o.e.n.Node", "cluster.name": "docker-cluster", "node.name": "795301fa4669", "message": "stopping ..." }
{"type": "server", "timestamp": "2020-05-03T10:12:51,443Z", "level": "INFO", "component": "o.e.n.Node", "cluster.name": "docker-cluster", "node.name": "795301fa4669", "message": "stopped" }
{"type": "server", "timestamp": "2020-05-03T10:12:51,444Z", "level": "INFO", "component": "o.e.n.Node", "cluster.name": "docker-cluster", "node.name": "795301fa4669", "message": "closing ..." }
{"type": "server", "timestamp": "2020-05-03T10:12:51,498Z", "level": "INFO", "component": "o.e.n.Node", "cluster.name": "docker-cluster", "node.name": "795301fa4669", "message": "closed" }
{"type": "server", "timestamp": "2020-05-03T10:12:51,502Z", "level": "INFO", "component": "o.e.x.m.p.NativeController", "cluster.name": "docker-cluster", "node.name": "795301fa4669", "message": "Native controller process has stopped - no new native processes can be started" }
```

处理上面问题所以在启动的时候添加参数 `-e "discovery.type=single-node"`运行



Elasticsearch 与我们关系型数据库的类比: 

```
Relational DB -> Databases -> Tables -> Rows -> Columns
Elasticsearch -> Indices -> Types -> Documents -> Fields
```

`Elasticsearch集群可以包含多个索引(indices)[数据库], 每一个索引可以包含多个类型(types)[表],每一个类型包含多个文档(documents)[行],然后每个文档包含多个字段(Fields)[列]`



**linux环境中启动还会提示如下错误: **

```
OpenJDK 64-Bit Server VM warning: Option UseConcMarkSweepGC was deprecated in version 9.0 and will likely be removed in a future release.
OpenJDK 64-Bit Server VM warning: INFO: os::commit_memory(0x00000000c5330000, 986513408, 0) failed; error='Not enough space' (errno=12)
```



**处理方法: **

```
// 查找jvm.options文件的位置
> find /var/lib/docker/overlay2/ -name "jvm*"

// 进入到jvm.options 路径中
> cd  /var/lib/docker/overlay2/768ee33cd17f55c4313bddcf3ba7671e1c17f0a1e890ad878abf55ca43ffe2bf/diff/usr/share/elasticsearch/config/

// 修改配置文件中的内容
-Xms1g
-Xmx1g

// 修改为
-Xms512m
-Xmx512m

// 退出保存重启Elasticsearch
```



在docker中下载Kibana: 

```
> docker pull kibana:7.6.2
```

启动连接到Elasticsearch:

```
> docker run -d --name kibana --link={es-containid} -p 5601:5601 kibana:7.6.2
```

启动之后访问`centos7-docker:5601`会提示下面错误信息: 

```
Kibana server is not ready yet
```

进入到Kibana容器, 编辑 `/config/kibana.yml` 配置文件: 

```
server.name: kibana
server.host: "0"
# 讲hosts 换成目标elasticsearch, 默认elasticsearch
elasticsearch.hosts: [ "http://node1:9200" ]
xpack.monitoring.ui.container.elasticsearch.enabled: true
```

修改完成之后重启容器, 在访问, 成功.



往Elasticsearch中添加数据: 

```
PUT http://localhost:9200/megacorp/employee/1
Accept: application/json
Cache-Control: no-cache
Content-Type: application/json

{ "first_name" : "John", "last_name" : "Smith", "age" : 25, "about" : "I love to go rock climbing", "interests": [ "sports", "music" ] }

```

查询数据: 

` GET http://localhost:9200/megacorp/employee/1`

集群健康: 

`GET /_cluster/health`

| 颜色   | 意义                                       |
| ------ | ------------------------------------------ |
| green  | 所有主要分片和复制分片均可用               |
| yellow | 所有主要分片可用, 但不是所有复制分片都可用 |
| red    | 不是所有的主要分片都可用                   |

**主要分片(primary shard):**

索引中的每个文档属于一个单独的主分片, 所以主分片的数量决定了索引最多能存储多少数据。

**复制分片(replica shard): **

复制分片只是主分片的一个副本,它可以防止硬件故障导致的数据丢失.同时可以提供读请求, 比如搜索或者从别地shard 取回文档。

`索引(index): 一个存储关联数据的地方, 一个用来指向一个或多个分片(shards)的逻辑命名空间`

`分片(shard): 一个最小级别'工作单元',只保存了索引中所有数据的一部分`



`默认情况下, 一个索引被分配5个主分片`

添加索引可以设置: 

```
PUT /blogs

{
	"settings": {
		"number_of_shards": 3,
		"number_of_replicas": 1
	}
}
```



**Elasticsearch集群配置: **

```
cluster.name: elasticsearch #集群的名称，同一个集群该值必须设置成相同的
node.name: es4 #该节点的名字
node.master: false #该节点有机会成为master节点
node.data: true #该节点可以存储数据
network.bind_host: 0.0.0.0 #设置绑定的IP地址，可以是IPV4或者IPV6
network.publish_host: 192.168.85.135 #设置其他节点与该节点交互的IP地址
network.host: 192.168.85.135 #该参数用于同时设置bind_host和publish_host
transport.tcp.port: 9500 #设置节点之间交互的端口号
transport.tcp.compress: true #设置是否压缩tcp上交互传输的数据
http.port: 9400 #设置对外服务的http端口号
http.max_content_length: 100mb #设置http内容的最大大小
http.enabled: true #是否开启http服务对外提供服务
discovery.zen.minimum_master_nodes: 2 #设置这个参数来保证集群中的节点可以知道其它N个有master资格的节点。官方推荐（N/2）+1
discovery.zen.ping_timeout: 120s #设置集群中自动发现其他节点时ping连接的超时时间
discovery.zen.ping.unicast.hosts: ["192.168.85.133:9300","192.168.85.133:9500","192.168.85.135:9300"] #设置集群中的Master节点的初始列表，可以通过这些节点来自动发现其他新加入集群的节点
http.cors.enabled: true  #跨域连接相关设置
http.cors.allow-origin: "*"  #跨域连接相关设置  
```

