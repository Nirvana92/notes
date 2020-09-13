性能优化: 

1. 合并&压缩
2. Lazy Queue
3. HiPE
4. Lazy + HiPE
5. 流控链

消息堆积的治理: 

1. 丢弃策略: 设置消息保留时间或者保留大小
2. Lazy Queue
3. 移花接木: 运行的集群 `cluster1` 发生严重堆积时通过`Shovel` 将数据迁移到备份集群`cluster2`. 消息堆积缓解的时候, 通过`Shovel`将消息通过`cluster2` 到`cluster1`.