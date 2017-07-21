title: 深入浅出mysql读书笔记-基础篇
date: 2014-11-21 16:54:07
tags:
- mysql
category:
- mysql
---

查看一个表的结构
```mysql 
mysql> desc cats; 
+-------+------------------+------+-----+---------+----------------+
| Field | Type             | Null | Key | Default | Extra          |
+-------+------------------+------+-----+---------+----------------+
| id    | int(10) unsigned | NO   | PRI | NULL    | auto_increment |
| name  | char(15)         | NO   |     | NULL    |                |
+-------+------------------+------+-----+---------+----------------+
2 rows in set (0.04 sec)

```
查看一个表的创建语句
```mysql 
mysql> show create table cats\G;
*************************** 1. row ***************************
       Table: cats
Create Table: CREATE TABLE `cats` (
  `id` int(10) unsigned NOT NULL AUTO_INCREMENT,
  `name` char(15) CHARACTER SET utf8 NOT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=8 DEFAULT CHARSET=latin1
1 row in set (0.00 sec)
```
修改字段的名字
```mysql 
mysql> alter table cats change `name` `cat_name` char(15);

```
分组统计后汇总
```mysql 

mysql> select kind,count(1) from cats group by kind with rollup;
+------+----------+
| kind | count(1) |
+------+----------+
|    1 |        1 |
|    2 |        2 |
| NULL |        3 |
+------+----------+

```

查询帮助文档
```mysql
mysql> ? grant
Name: 'GRANT'
Description:
Syntax:
GRANT
    priv_type [(column_list)]
      [, priv_type [(column_list)]] ...
    ON [object_type] priv_level
    TO user_specification [, user_specification] ...
    [REQUIRE {NONE | ssl_option [[AND] ssl_option] ...}]
    [WITH with_option ...]

```

查询一个bit类型的字段
```
mysql> select flag+0,bin(flag),hex(flag) from cats where id = 10;
+--------+-----------+-----------+
| flag+0 | bin(flag) | hex(flag) |
+--------+-----------+-----------+
|     50 | 110010    | 32        |
+--------+-----------+-----------+
1 row in set (0.00 sec)

```

datetime 类型和dateline类型 的区别 
datetime 是不分时区的 存进去是啥就是啥
dateline 是分时区的 取出时会按时区进行转换 另外它的默认值是CURRENT_TIMESTAMP 如果插入null则转为当前时间 并且一个表中只能有一个列有这个默认值 

<=> 可以正确比较null值
```mysql
select null = null;
+-------------+
| null = null |
+-------------+
|        NULL |
+-------------+
1 row in set (0.00 sec)

select null<=>null;
+-------------+
| null<=>null |
+-------------+
|           1 |
+-------------+
1 row in set (0.00 sec)

```

将ip地址转换为数字
```mysql
mysql> select INET_ATON('192.168.0.1');
+--------------------------+
| INET_ATON('192.168.0.1') |
+--------------------------+
|               3232235521 |
+--------------------------+
1 row in set (0.00 sec)

mysql> select INET_NTOA(3232235521);
+-----------------------+
| INET_NTOA(3232235521) |
+-----------------------+
| 192.168.0.1           |
+-----------------------+
1 row in set (0.00 sec)

```





