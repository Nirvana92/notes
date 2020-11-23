#### Redo Log

> redo log 作用: https://dev.mysql.com/doc/refman/5.7/en/glossary.html#glos_crash_recovery

> Redo log 是基于磁盘的数据结构。在崩溃恢复期间用于纠正不完整事务写入的数据。默认情况下, Redo-log 在磁盘上由两个名为ib_logfile0和ib_logfile1。
>
> mysql以循环的方式写入redo-log。通过增加LSN的值表示。

LSN参考: `https://dev.mysql.com/doc/refman/5.7/en/glossary.html#glos_lsn`

**更改InnoDB redo-log 文件的数量或大小**

`innodb_log_file_size`: 更改日志文件的大小

`innodb_log_files_in_group`: 增加日志文件的数量

如果InnoDB检测到 `innodb_log_file_size`与redo-log大小不同。它将编写日志检查点[checkpoint]。关闭并删除旧的日志文件，以请求的大小创建新的日志文件，然后使用新的日志文件。

**Group commits for redo log flushing**

> 提交事务之前刷新事务redo-log。使用组提交功能将多个此类刷新请求分组到一起。避免每次提交都进行一次刷新。提供了吞吐量。



提交数据[插入、更新]的过程: 

提交的数据 -> change buffer -> redo log -> undo log -> disk file

