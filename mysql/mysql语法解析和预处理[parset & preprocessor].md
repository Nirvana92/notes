词法解析: 

比如: 

```sql
select name from user where id=1;
```

会被打碎成8个符号。



语法解析: 会对sql做一些语法检查，比如单引号有没有关闭。然后根据mysql定义的语法规则，生成一个解析树[select_lex]

![img](https://img2018.cnblogs.com/i-beta/1511203/201912/1511203-20191230230616541-1826981645.png)



预处理器[解析sql阶段]: 它会检查表和列表是否存在，检查名字和别名，保证没有歧义。

```sql
show status like 'last_query_cost';
```

启用优化器的追踪[默认关闭]: 

```sql
show variables like '%optimizer_trace%';
set optimizer_trace='enabled=on';
```

开启会消耗性能, 它会把优化分析的结果写到表中。



buffer pool: 默认大小是128M[134217728字节]

change buffer :

```sql
# change buffer 占buffer pool 的比例, 默认 25%
show variables like '%innodb_change_buffer_max_size%'; 
```

adaptive hash index: 

redo log buffer: 



