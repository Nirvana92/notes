InnoDB系统参数: `https://dev.mysql.com/doc/refman/5.7/en/innodb-parameters.html`

InnoDB插入调优: `https://blog.51cto.com/7737197/1664565`

#### 优化sql语句

##### 优化select 语句

- 如果最佳索引没有超过表行的30%, 默认认为全表扫描效果更佳

