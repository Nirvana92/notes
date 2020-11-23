#### Undo-log

> 撤销日志是与单个读写事务关联的撤销日志记录的集合。InnoDB最大支持128个回滚段。其中32个分配给临时表空间。回滚段可以通过 innodb_rollback_segments 设置
>
> 
>
> ![截屏2020-05-30 09.37.03](/Users/mac/Documents/笔记/mysql/截屏2020-05-30 09.37.03.png)

一个事务最多可以分配四个撤销日志： 

- insert用户定义表上的操作
- update/delete 用户定义表上的操作
- insert 用户定义的临时表上的操作
- update/delete 用户定义的临时表上的操作

如果每个事务执行任一种操作[insert or update or delete]，并发读-写事务支持数目: 

```
(innodb_page_size/16)*(innodb_rollback_segments-32)
```

每个事务包含的操作[insert and (update or delete)]，则支持的并发读-写事务数目: 

```
(innodb_page_size/16/2)*(innodb_rollback_segments-32)
```

如果一个事务包含insert 在临时表，则支持的并发读-写事务数目: 

```
(innodb_page_size/16)*32
```

如果一个事务包含[insert and (update or delete)], 则支持的并发读写事务数目: 

```
(innodb_page_size/16/2)*32
```

