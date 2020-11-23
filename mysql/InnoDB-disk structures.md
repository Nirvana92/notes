#### 表[tables]

查看默认存储引擎: `select @@default_storage_engine`

建表手动指定存储引擎: 

```sql
CREATE TABLE t1 (a INT, b CHAR (20), PRIMARY KEY (a)) ENGINE=InnoDB;
```

存储引擎为InnoDB 的表, 它的索引可以在`system tablespace`/`file-per-table tablespace`/`general tablespace`中创建。



#### 索引[indexes]



#### 表空间[tablespaces]

表空间参考: `https://dev.mysql.com/doc/refman/5.7/en/innodb-tablespace.html`

- `The System Tablespaces`
- `File-Pre-Table Tablespaces`
- `General Tablespaces`
- `Undo Tablespaces`
- `The Temporary Tablesapce`

##### 系统表空间[system tablespace]

> 系统表空间是 InnoDB data dictionary, doublewrite buffer, change buffer, undo logs 存储的地方。如果表在系统表空间中创建, 也可能会包含表数据[table data]和索引数据[index data]

系统表空间可以有一到多个数据文件。默认是一个存储在数据路径下名为`ibdata1`的文件。系统表空间的大小和序号可以通过 `innodb_data_file_path`设置。参考: `https://dev.mysql.com/doc/refman/5.7/en/innodb-init-startup-configuration.html#innodb-startup-data-file-configuration`



mysql 数据文件存储路径配置: `datadir; select @@datadir;`

**调整系统表空间大小: **

> 增加系统表空间大小最简单的方法是将其配置为自动扩展。可以在 innodb_data_file_path 中设置 autoextend

例如: 

```sql
innodb_data_file_path=ibdata1:10M:autoextend
```

当设置 `autoextend`时, 自动

可以通过`innodb_autoextend_increment`[单位 M]控制增量大小。调大可以减少增量时候的mysql停顿。



还可以通过增加另一个数据文件来增加系统表空间的大小。

假设数据文件增长到988M, 并制定新的50M自动扩展数据文件的设置: 

```sql
innodb_data_file_path = /ibdata/ibdata1:988M;/disk2/ibdata2:50M:autoextend
```



**注意: **

> 不能通过更改大小属性增加现有系统表空间数据文件的大小。例如，在启动服务器时, 将 innodb_data_file_path 从 ibdata1:10M:autoextend 修改为 ibdata1:12M:autoextend 为产生如下错误: 

```sql
[ERROR] [MY-012263] [InnoDB] The Auto-extending innodb_system 
data file './ibdata1' is of a different size 640 pages (rounded down to MB) than 
specified in the .cnf file: initial 768 pages, max 0 (relevant if non-zero) pages!
```

InnoDB 页面大小通过`innodb_page_size`配置, 默认 16384字节。

**减少InnoDB系统表空间的大小**

1. 使用mysqldump转储所有InnoDB表。

   ```sql
   SELECT TABLE_NAME from INFORMATION_SCHEMA.TABLES WHERE TABLE_SCHEMA='mysql' and ENGINE='InnoDB';
   ```

   

2. 停止服务器

3. 删除所有现有的表空间文件[*.ibd], 包括ibdata和ib_log 文件。不要忘记删除mysql schema 中的 *.idb文件。

4. 删除InnoDB所有的.frm 文件。

5. 为新系统表空间配置数据文件。

6. 重启服务器

7. 倒入转储文件。

> 为避免使用大型系统表空间, 考虑对数据使用 file-per-table tablespace。file-per-table tablespace 时默认表空间类型。与系统表空间不同, 磁盘空间在截断或删除在每个file-per-table tablespace 中创建表后会返回给操作系统。参考: https://dev.mysql.com/doc/refman/5.7/en/innodb-file-per-table-tablespaces.html

##### 单表文件表空间 [file-per-table tablespace]

> file-per-table tablespace 包含单个InnoDB 表的数据和索引。并存储在文件系统自己的数据文件中。

**file-per-table tablespace 配置**

> InnoDB 默认情况下, 创建表在 file-per-table tablespaces.可以通过 innodb_file_per_table 配置.禁用该配置则会在 system tablespace 中创建表。

**file-per-table tablespace 数据文件**

.idb 在mysql数据目录下的每个数据库中。.idb 文件以表(table_name.ibd)命名。

**file-per-table tablespace 的优势**

和 system tablespace /general tablespace 相比, file-per-table tablespace 有如下优势: 

- 删除在 file-per-table tablespace 创建的表后, 磁盘空间将返回操作系统。
- TRUNCATE TABLE 在 file-per-table tablespace 上执行, 性能会更好。

##### 通用表空间[General Tablespaces]

> 通用表空间时InnoDB 使用create tablespace 语法创建的共享表空间。

**功能**

- 类似系统表空间, 是共享表空间。可以存储多个表的数据。

**创建通用表空间**

```sql
CREATE TABLESPACE tablespace_name
    ADD DATAFILE 'file_name'
    [FILE_BLOCK_SIZE = value]
        [ENGINE [=] engine_name]
```

在mysql数据目录之外创建通用表空间时，会在mysql 数据目录中创建一个 .isl文件。

**将表添加到通用表空间**

```sql
CREATE TABLE t1 (c1 INT PRIMARY KEY) TABLESPACE ts1;
```

alter table: 

```sql
ALTER TABLE t2 TABLESPACE ts1;
```

