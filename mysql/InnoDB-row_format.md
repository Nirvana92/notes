在InnoDB 存储引擎中, **数据**`[聚簇索引]`和**索引**都存储在b+tree中.

可变长度类型`[varchar, varbinary, blob, text]`的字段太长无法存储在b+tree 页面。需要单独分配在磁盘页面上, 这些页面称为`[溢出页面 overflow pages]`,这些列称为`[页外列 off-page columns]`。这样的列都有自己的溢出页面的列表。根据列的长度, 所有或可变长度列值的前缀都存储在b+tree中。



InnoDB文件格式: 

参考: `https://dev.mysql.com/doc/refman/5.7/en/innodb-file-format.html`

- `Antelope:` 原始InnoDB文件格式。支持`COMPACT`和`REDUNDANT`行格式

- `Barracuda:` 最新的文件格式。支持所有InnoDB行格式。InnoDB文件格式设置不适用于存储在 常规表空间中的表`[general tablespaces]`。常规表空间参考: `https://dev.mysql.com/doc/refman/5.7/en/general-tablespaces.html`

  

配置参数: 

`innodb_file_format:`默认值改为 `Barracuda`, 以前默认值为`Antelope`

`innodb_large_prefix: `默认值改为`on`, 以前默认值为`off`



表空间参考: `https://dev.mysql.com/doc/refman/5.7/en/innodb-tablespace.html`

- `The System Tablespaces`
- `File-Pre-Table Tablespaces`
- `General Tablespaces`
- `Undo Tablespaces`
- `The Temporary Tablesapce`



以下文件格式配置参数已经弃用, 将来版本可能会删除: 

`innodb_file_format`, `innodb_file_format_check`, `innodb_file_format_max`, `innodb_large_prefix`



InnoDB支持的`row_format:` 

- `REDUNDANT:` 兼容旧版本的mysql。支持InnoDB的文件格式[Antelope, Barracuda]。将变长字段`[varchar, varbinary, blob, text]`前768个字节存储在b+tree 索引记录中。其余的存储在溢出页上。

- `COMPACT:` 与``REDUNDANT` 相比, `COMPACT` 减少行存储空间近20%。但是某些操作增加了cpu的消耗。支持InnoDB的文件格式[Antelope, Barracuda]。将变长字段`[varchar, varbinary, blob, text]`前768个字节存储在b+tree 索引记录中。其余的存储在溢出页上。

- `DYNAMIC: `拥有`COMPACT`一样的存储特性, `DYNAMIC`将可变长度列值完全存储在页外。聚簇索引就仅包含指向溢出页的20字节指针。数据较小的时候比如 40字节将会存储在行。支持索引键前缀可达3072字节。可由`innodb_large_prefix`控制。`DYNAMIC`行格式可用在 `system tablespace`/`file-per-table tablespaces`/`general tablespaces`。存储在system tablespace, 设置 `innodb_file_per_table=OFF`

- `COMPRESSED: `压缩行格式。和`DYNAMIC`提供相同的存储特性和功能。但增加了对表和索引数据压缩的支持。`KEY_BLOCK_SIZE`选项控制在聚簇索引中存储多少列数据，以及在溢出页面上放置多少列数据。参考: `https://dev.mysql.com/doc/refman/5.7/en/innodb-compression.html`该行格式支持索引键的前缀可达3072个字节。可以通过`innodb_large_prefix`变量控制。

  

![截屏2020-05-29 20.13.03](/Users/mac/Documents/笔记/mysql/截屏2020-05-29 20.13.03.png)



定义表的 `row_format`: 默认的`row_format`通过`innodb_default_row_format`变量控制。默认值是 `dynamic`。

也可以在建表的时候手动设置: 

```sql
CREATE TABLE t1 (c1 INT) ROW_FORMAT=DYNAMIC;
```

