#### chagne buffer

> https://www.sohu.com/a/322957463_178889

**什么是InnoDB的写缓冲？**

在MySQL5.5之前，叫插入缓冲(insert buffer)，只针对insert做了优化；现在对delete和update也有效，叫做写缓冲(change buffer)。

它是一种应用在**非唯一普通索引页**(non-unique secondary index page)不在缓冲池中，对页进行了写操作，并不会立刻将磁盘页加载到缓冲池，而仅仅记录缓冲变更(buffer changes)，等未来数据被读取时，再将数据合并(merge)恢复到缓冲池中的技术。写缓冲的**目的**是降低写操作的磁盘IO，提升数据库性能。



**为什么写缓冲优化，仅适用于非唯一普通索引页呢？**

InnoDB里，聚集索引(clustered index)和普通索引(secondary index)的异同，《1分钟了解MyISAM与InnoDB的索引差异》有详尽的叙述，不再展开。

如果索引设置了唯一(unique)属性，在进行修改操作时，InnoDB必须进行唯一性检查。也就是说，索引页即使不在缓冲池，磁盘上的页读取无法避免(否则怎么校验是否唯一？)，此时就应该直接把相应的页放入缓冲池再进行修改，而不应该再整写缓冲这个幺蛾子。