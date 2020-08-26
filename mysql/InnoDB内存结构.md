#### Buffer pool

前面是最近访问过的新页面[年轻]的子列表

在末尾，是最近访问的旧页面的字列表

`innodb_old_blocks_pct: `控制LRU 列表中`old`块的百分比。默认37,对应于3/8的比率。值的范围是 5~95。

`innodb_old_blocks_time: `指定第一次访问页面后的时间窗口[ms单位]，在该时间窗口内可以访问页面也不将其移动到LRU列表的最前面, 默认1000。增加此值会使越来越多的块从缓冲池中更快的老化。

![innodb-buffer-pool-list](/Users/mac/Documents/笔记/mysql/innodb-buffer-pool-list.png)

可以通过 `show engine innodb status;`查看缓冲池统计信息。

##### 配置InnoDB缓冲池预读

> 预读请求使IO请求异步预取多个页面缓冲池，期待这些页面将很快被使用。

InnoDB使用两种预读算法来提高IO性能: 

**线性预读: ** 根据顺序访问的缓冲池中的页面来预测很快可能需要哪些页面。可以通过`innodb_read_ahead_threshold`来控制InnoDB检测顺序页面访问模式时的敏感度。可以设置0~64之间的任何值。

**随机预读: **它根据缓冲池中已有的页面来预测何时可能需要改页面，而不管这些页面的读取顺序如何。启用此功能，请将配置变量`innodb_random_read_ahead=ON`。

通过`show engine innodb status;`命令显示统计信息。

参数参考: `https://dev.mysql.com/doc/refman/5.7/en/server-status-variables.html#statvar_Innodb_buffer_pool_read_ahead`

`innodb_buffer_pool_read_ahead`: 从页读入到InnoDB buffer pool 后台的 read_ahead 线程数。

`innodb_buffer_pool_read_ahead_evicted`: 

`innodb_buffer_pool_read_ahead_rnd`: 

##### 缓冲池[buffer pool]配置

`innodb_buffer_pool_instances:` 配置缓冲池实例。1~64。下面配置超过 1G的时候, 该参数才能生效。

`innodb_buffer_pool_size:`调整buffer pool的大小。



配置缓冲池[buffer pool]刷新: `https://dev.mysql.com/doc/refman/5.7/en/innodb-buffer-pool-flushing.html`



#### Change Buffer

> change buffer 是一种特殊的数据结构。当二级索引页不在buffer pool 中时。他们会缓存这些更改。当页面通过其他读取操作加载到buffer pool 中时。可能由insert/update/delete操作导致缓冲更改，将在以后合并。



`innodb_change_buffering: `

all: 默认值, 缓冲区插入/删除标记操作

none: 不需要缓冲任何操作

inserts: 缓冲插入操作

deletes: 缓冲删除标记操作

changes: 缓冲插入和删除标记操作

purges: 缓冲在后台发生的物理删除操作。



`innodb_change_buffer_max_size: `允许将更改缓冲区的最大大小配置.为buffer pool的百分比。默认为25, 最大设置50。

适合场景: 

- 数据库大部分时非唯一索引
- 业务是写多读少，或者不是写后立刻读取

#### Adaptive Hash Index

自适应hash索引。Innodb 自己操作的。可以提高效率。

通过`innodb_adaptive_hash_index`变量启用。

#### Log Buffer

> 日志缓冲区是存储区域, 用于保存要写入磁盘上的日志文件的数据。日志缓冲区大小由 innodb_log_buffer_size 定义。默认16M。日志缓冲区的内容会定期刷新到磁盘。
>
> innodb_flush_log_at_trx_commit: 控制如何将日志缓冲区的内容写入并刷新到磁盘。

**innodb_flush_log_at_trx_commit: **

**0:**每秒写入一次日志并将其刷新到磁盘, 尚未刷新日志的事务可能会在崩溃中丢失

**1:**完全符合acid[默认值]

**2:** 每次事务提交后写入日志, 并每秒刷新一次到磁盘, 尚未刷新日志的事务可能在崩溃中丢失

> innodb_flush_log_at_timeout: 控制日志刷新频率。

**innodb_flush_log_at_timeout: **

write and flush log every n seconds. 默认每秒刷新一次。