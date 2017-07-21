title: 源码安装mysql
date: 2014-12-30 18:27:35
tags:
- mysql
category:
- mysql
---

mysql 源码包下载地址
https://github.com/mysql/mysql-server

参考文档
[http://dev.mysql.com/doc/refman/5.5/en/installing-source-distribution.html](http://dev.mysql.com/doc/refman/5.5/en/installing-source-distribution.html)



详细的步骤见官方文档即可 只记录几个关键步骤

安装依赖软件
yum  install ncurses-devel cmake 

cmake时的主要编译参数
cmake -DCMAKE_INSTALL_PREFIX=/usr/local/mysql -DMYSQL_DATADIR=/usr/local/mysql/data -DSYSCONFDIR=/etc -DWITH_INNOBASE_STORAGE_ENGINE=1 -DMYSQL_UNIX_ADDR=/var/lib/mysql/mysql.sock -DMYSQL_TCP_PORT=3306 -DENABLED_LOCAL_INFILE=1 -DWITH_PARTITION_STORAGE_ENGINE=1 -DEXTRA_CHARSETS=all -DDEFAULT_CHARSET=utf8 -DDEFAULT_COLLATION=utf8_general_ci

默认的时候是没有innodb引擎的 所以加入 WITH_INNOBASE_STORAGE_ENGINE
ENABLED_LOCAL_INFILE 选项可以支持在客户端以load data 语句加载数据
WITH_PARTITION_STORAGE_ENGINE 加入对分区的支持

scripts/mysql_install_db --basedir=/usr/local/mysql --datadir=/usr/local/mysql/data --user=mysql

还有个注意的地方是如果开启了binlog 他默认会生产到MYSQL_DATADIR所在的目录 这个文件生产后属主可能不是mysql 如要重新chown  一下


