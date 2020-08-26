mysql数据库备份脚本: 

```shell
#! /bin/bash

today=$(date "+%Y%m%d%H%M%S")
echo $today

/home/gzm/mysql-5.7.28-linux-glibc2.12-x86_64/bin/mysqldump -uroot -piOPlHV0i15F8G2Nw p
ig > /home/gzm/mysqlbak/pig.sql.bak-$today

/home/gzm/mysql-5.7.28-linux-glibc2.12-x86_64/bin/mysqldump -uroot -piOPlHV0i15F8G2Nw x
xl-job > /home/gzm/mysqlbak/xxl-job.sql.bak-$today
```

备份的定时任务[crontab]: 

```shell
0 0 * * 0  sh /home/gzm/mysqlbak/mysqlbak.sh
```

匹配文件名并删除: 

```
> rm $(find . -name "xxl-job.sql.bak*")
```

