#### buffer pool

> 参考: https://dev.mysql.com/doc/refman/5.7/en/glossary.html#glos_buffer_pool

内存中缓存`InnoDB`的数据和索引的部分。为了更快的执行读取数据的操作。通过LRU算法。保证队列中的数据更新策略。