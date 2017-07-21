title: coreseek 从安装到使用总结
date: 2014-12-09 14:41:58
tags:
- careseek
category:
- linux
---
参考文章
[http://www.cnblogs.com/semonxv/p/3816376.html](http://www.cnblogs.com/semonxv/p/3816376.html)
[http://www.cnblogs.com/yjf512/p/3581869.html](http://www.cnblogs.com/yjf512/p/3581869.html)
[http://www.cnblogs.com/xingmeng/p/3723087.html](http://www.cnblogs.com/xingmeng/p/3723087.html)
[http://www.ibm.com/developerworks/cn/opensource/os-sphinx/](http://www.ibm.com/developerworks/cn/opensource/os-sphinx/)
[http://www.coreseek.cn/docs/coreseek_4.1-sphinx_2.0.1-beta.html](http://www.coreseek.cn/docs/coreseek_4.1-sphinx_2.0.1-beta.html)
[http://sphinxsearch.com/wiki/doku.php?id=sphinx_chinese_tutorial](http://sphinxsearch.com/wiki/doku.php?id=sphinx_chinese_tutorial)
[http://www.imsiren.com/archives/766](http://www.imsiren.com/archives/766)

简单的说sphinx就是从某个数据源取过来数据（最常见的就是数据库） 生成索引文件（很类似数据库的索引） 然后利用索引实现快速搜索 

#安装
安装mmseg 这是个中文分词库其实如果你不用分词就按中文汉字搜都不用安这个
```bash
wget http://www.coreseek.cn/uploads/csft/4.0/coreseek-4.1-beta.tar.gz
tar zxvf coreseek-4.1-beta.tar.gz

cd coreseek-4.1-beta
cd mmseg-3.2.14
./bootstrap
./configure --prefix=/usr/local/mmseg3
make && make install
```
中间出现的错误
error: possibly undefined macro: AM_PROG_LIBTOOL
办法 
yum install libtool

libtool: Version mismatch error.  This is libtool 2.2.6b
办法
直接复制 /usr/local/bin/litool到mmseg-3.2.14目录下，覆盖原来的libtool，然后重新make


安装coreseek 这个是核心软件我们的整个服务都是基于它的
```bash
cd /usr/local/src/coreseek-4.1-beta
./buildconf.sh
./configure --prefix=/usr/local/coreseek --with-mysql-includes=/usr/include/mysql/ --with-mysql-libs=/usr/lib64/mysql/ --with-mmseg=/usr/local/mmseg/ --with-mmseg-includes=/usr/local/mmseg/include/mmseg/ --with-mmseg-libs=/usr/local/mmseg/lib/
make
make install
```
这里的mysql 是yum安装的所以分开指定mysql的头文件 和库文件

安装完成之后/usr/local/coreseek/bin 目录下是它的几个主要可执行程序
indexer 这个程序是用来生成索引文件的
searchd 这个是对外提供接口服务的守护进程
search  这个是用来在命令行调试的不用在实际开发中
还有两个是导出用的还没研究

#配置
mmseg基本不用配置 我也不会配置

进coreseek的配置文件目录 
/usr/local/coreseek/etc
这里面两个自带的配置文件 一个是带全部注释的 一个是精简版的配置文件
我们改造个精简版的来说下基本配置

先在mysql建好一个文章表 news 测试用
```bash
+----------+----------------------+------+-----+---------+----------------+
| Field    | Type                 | Null | Key | Default | Extra          |
+----------+----------------------+------+-----+---------+----------------+
| id       | int(11)              | NO   | PRI | NULL    | auto_increment |
| title    | varchar(255)         | NO   |     | NULL    |                |
| content  | text                 | NO   |     | NULL    |                |
| catid    | smallint(5) unsigned | NO   |     | NULL    |                |
| dateline | datetime             | NO   |     | NULL    |                |
+----------+----------------------+------+-----+---------+----------------+

```
```bash
#
# Minimal Sphinx configuration sample (clean, simple, functional)
#

source src1
{
        type                    = mysql

        sql_host                = localhost
        sql_user                = root
        sql_pass                =
        sql_db                  = news
        sql_port                = 3306  # optional, default is 3306
        sql_query_pre           = set names utf8
        sql_query               = \
                SELECT id,catid , UNIX_TIMESTAMP(dateline) AS dateline, title, content \
                FROM news

        sql_attr_uint           = catid
        sql_attr_timestamp      = dateline
        sql_field_string        = title
        sql_field_string        = content
        sql_query_info          = SELECT * FROM news WHERE id=$id
}


index test1
{
        source                  = src1
        path                    = /usr/local/coreseek/var/data/test1
        docinfo                 = extern
        charset_type            = zh_cn.utf-8 #设置数据编码为UTF8
        ngram_len               = 0
        charset_dictpath        = /usr/local/mmseg/etc/
        #html_strip             =1 #去掉HTML标签
}



indexer
{
        mem_limit               = 32M
}


searchd
{
        listen                  = 9312
        listen                  = 9306:mysql41
        log                     = /usr/local/coreseek/var/log/searchd.log
        query_log               = /usr/local/coreseek/var/log/query.log
        read_timeout            = 5
        max_children            = 30
        pid_file                = /usr/local/coreseek/var/log/searchd.pid
        max_matches             = 1000
        seamless_rotate         = 1
        preopen_indexes         = 1
        unlink_old              = 1
        workers                 = threads # for RT to work
}

```

主要分四大块
>source
数据源的配置 indexer在生成索引的时候就是根据这里的配置去读取mysql
sql_query_pre 这个是在主查询之前进行的sql操作 一般是设置字符集什么的
sql_query 这个就是核心的查询了 用来从数据库获取数据
sql_attr_* 这个是属性 他是用来过滤用的相当于where的条件（把sphinx看成一种数据库就行）但是他是不能被全文检索的
sql_field_* 这个是提供全文检索的字段 

>index
这个是索引的配置选项
path 指定索引文件生成目录
docinfo 表示属性信息的存储方式 none 不存贮 inline和索引存在一起 docinfo单独存放
charset_type 这个一定要设置成utf-8
ngram_len 和 ngram_chars 这个是对未分词的中文用的 因为咱们配置了分词 这里要把ngram_len=0 设置成关闭 表示消原有的一元字符切分模式，不使其对中文分词产生干扰；
charset_table 接受的字符表和大小写转换规则 这个也注释掉 交给mmseg去做这些 Coreseek 提供的MMseg分词法内置了可接受的字符表，并且用户不可修改。当启用分词功能时，自动开启。
charset_dictpath 这个是核心配置啦 指定分词词典文件的路径

>indexer
索引的一些配置项

>searchd
守护进程的配置项
listen的格式 listen = ( address ":" port | port | path ) [ ":" protocol ]
9312端口提供的是api方式调用
9306端口指定了协议是mysql41 也就是可以像操作mysql一样写sql语句来执行访问 具体见官方SphinxQL说明

#使用
先用indexer生成索引
 /usr/local/coreseek/bin/indexer -c /usr/local/coreseek/etc/sphinx.conf --all --rotate
 如果是第一次生成不用加rotate 如果searchd服务已经运行则必须加rotate 意思是先生成一个临时索引文件 生成完毕后将正式索引变更为这个文件
用search 测试一个搜索


开启searchd服务
/usr/local/coreseek/bin/searchd -c /usr/local/coreseek/etc/sphinx.conf
