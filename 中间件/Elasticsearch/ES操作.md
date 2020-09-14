索引一个文档: 

新建索引: 

```shell
POST /{index}/{type}/{id}
{
	"field": "value",
	....
}
```

比如: 

```shell
POST /website/blog/1
{
	"title": "My First blog entry",
	"text": "Just trying this out...",
	"date": "2020-09-14"
}
```

`?pretty=true`: 参数的作用. 在查询的后面添加 `pretty=true`参数格式化json输出。美化输出。

第二种方法: `PUT`

使用方式和`POST`的使用方式一样. 可添加可修改。

```shell
PUT /{index}/{type}/{id}
{
	"field": "value",
	....
}
```

如果想使用自定义的 _id, 必须告诉es应该在_ _index, _type, _id 三者都不同的时候才接受请求: 

```shell
> PUT /app/_create/3
{
  "name": "小明",
  "age": 1
}
```

如果存在会提示如下信息: 

```shell
{
  "error" : {
    "root_cause" : [
      {
        "type" : "version_conflict_engine_exception",
        "reason" : "[3]: version conflict, document already exists (current version [1])",
        "index_uuid" : "uZ1pO-kfSSq8YexVgBvRcQ",
        "shard" : "0",
        "index" : "app"
      }
    ],
    "type" : "version_conflict_engine_exception",
    "reason" : "[3]: version conflict, document already exists (current version [1])",
    "index_uuid" : "uZ1pO-kfSSq8YexVgBvRcQ",
    "shard" : "0",
    "index" : "app"
  },
  "status" : 409
}

```

乐观并发控制: 

```shell
PUT /app/_doc/3?if_seq_no=4&if_primary_term=1
{
  "name": "乐观锁控制名",
  "age": 1
}
```



查询数据: 

```shell
> GET /website/blog/2
```

返回数据: 

```shell
{
  "_index" : "website",
  "_type" : "blog",
  "_id" : "2",
  "_version" : 1,
  "_seq_no" : 2,
  "_primary_term" : 1,
  "found" : true,
  "_source" : {
    "title" : "My second blog entry",
    "text" : "测试往es中添加数据",
    "date" : "2020-09-15"
  }
}
```

删除文档数据: 

```shell
> DELETE /website/blog/1
```

**es 中每个文档都有版本号, 每当文档有变化[包括删除]都会使 _version 增加。**

**自增ID:** 如果我们数据没有自然id, es可以自动为我们生成。

```shell
PUT /app/user/
{
  "name":"Nirvana",
  "age": 12,
  "birth": "2020-09-14 22:37:53",
  "phoneNumber": "15168479052"
}
```

操作返回结果: 

```shell
{
  "_index" : "app",
  "_type" : "user",
  "_id" : "vl8QjXQB6GekCths-JeK",			# 变成自动生成id
  "_version" : 1,
  "result" : "created",
  "_shards" : {
    "total" : 2,
    "successful" : 1,
    "failed" : 0
  },
  "_seq_no" : 2,
  "_primary_term" : 1
}
```

检索文档的一部分: 

```shell
> GET /app/user/1?_source=name,age
```

返回结果: 

```shell
{
  "_index" : "app",
  "_type" : "user",
  "_id" : "1",
  "_version" : 2,
  "_seq_no" : 1,
  "_primary_term" : 1,
  "found" : true,
  "_source" : {
    "name" : "Nirvana",
    "age" : 12
  }
}
```

如果只想得到 _source 字段而不要其他的元数据, 可以执行如下命令: 

```shell
> GET /app/user/1/_source
```

返回结果: 

```shell
{
  "name" : "Nirvana",
  "age" : 12,
  "birth" : "2020-09-14 22:37:53",
  "phoneNumber" : "15168479052"
}
```

检查文档是否存在: 

```shell
# 执行 curl -i -XHEAD http://localhost:9200/app/user/1 提示过期, 应该执行下面语句
> curl -i -XHEAD http://localhost:9200/app/_doc/1
```

全部搜索: 

```shell
> GET /app/_search
```

分页: 

`size: `结果数, 默认10

`from: `跳过开始的结果数, 默认0

```shell
> GET /app/_search?size=1&from=2
```

简易模糊搜索: 

```shell
> GET /_all/_search?q=name:Nirvana
```

返回包含`Nirvana`字符的所有文档的简单搜索: 

```shell
> GET /_all/_search?q=Nirvana
```

根据日期返回搜索: 

```shell
POST /_all/_search
{
  "query": {
    "range": {
      "birth": {
        "gte": "2020-09-10"
      }
    }
  }
}
```

根据时间查询: 

```shell
GET /_search?q=birth:2020-09-14
```

