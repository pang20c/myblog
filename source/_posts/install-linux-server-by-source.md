title: install linux server by source
date: 2015-01-16 17:19:23
tags:
- linux
category:
- linux
---
安装第三方yum源
wget http://www.atomicorp.com/installers/atomic
sh ./atomic

安装依赖软件
yum -y install gcc gcc-c++ autoconf libjpeg libjpeg-devel libpng libpng-devel freetype freetype-devel libxml2 libxml2-devel zlib zlib-devel glibc glibc-devel glib2 glib2-devel bzip2 bzip2-devel ncurses ncurses-devel curl curl-devel e2fsprogs e2fsprogs-devel krb5 krb5-devel libidn libidn-devel openssl openssl-devel  cmake bison  libmcrypt  libmcrypt-devel libtool libtool-ltdl libtool-ltdl-devel openldap openldap-devel


##nginx安装

下载[pcre](http://sourceforge.net/projects/pcre/files/pcre/8.35/pcre-8.35.tar.gz/download) 源码包并解压
tar -zxvf pcre-8.35.tar.gz

添加www用户
useradd -Mrs /sbin/nologin www

下载nginx
wget http://nginx.org/download/nginx-1.7.9.tar.gz
tar -zxvf nginx-1.7.9.tar.gz
cd nginx-1.7.9

开始编译安装nginx
 ./configure --user=www --group=www --prefix=/usr/local/nginx --with-http_stub_status_module --with-http_ssl_module --with-pcre=/usr/local/src/pcre-8.35  --with-http_realip_module
 make && make install

 ##mysql5.5安装
useradd -Mrs /sbin/nologin mysql
unzip mysql-server-5.5.zip
cd mysql-server-5.5

cmake -DCMAKE_INSTALL_PREFIX=/usr/local/mysql -DMYSQL_DATADIR=/usr/local/mysql/data -DINSTALL_LIBDIR=/usr/lib -DSYSCONFDIR=/etc -DWITH_INNOBASE_STORAGE_ENGINE=1 -DMYSQL_UNIX_ADDR=/var/lib/mysql/mysql.sock -DMYSQL_TCP_PORT=3306 -DENABLED_LOCAL_INFILE=1 -DWITH_PARTITION_STORAGE_ENGINE=1 -DEXTRA_CHARSETS=all -DDEFAULT_CHARSET=utf8 -DDEFAULT_COLLATION=utf8_general_ci

make 
make install 

cd /usr/local/mysql/

chown -R mysql:mysql .

scripts/mysql_install_db --user=mysql --basedir=/usr/local/mysql/ --datadir=/usr/local/mysql/data/

cp support-files/my-medium.cnf /etc/my.cnf

bin/mysqld_safe --user=mysql &  启动mysql

##php 5.6.4 安装

cp -frp /usr/lib64/libldap* /usr/lib/
ldconfig -v 
./configure --prefix=/usr/local/php --with-config-file-path=/usr/local/php/etc --with-mysql --with-mysqli=/usr/local/mysql/bin/mysql_config --with-iconv-dir=/usr/local --with-freetype-dir --with-jpeg-dir --with-png-dir --with-zlib --with-libxml-dir=/usr --enable-xml --disable-rpath --enable-discard-path --enable-safe-mode --enable-bcmath --enable-shmop --enable-sysvsem --enable-inline-optimization --with-curl --with-curlwrappers --enable-mbregex --enable-fastcgi --enable-fpm --enable-force-cgi-redirect --enable-mbstring --with-mcrypt --with-gd --enable-gd-native-ttf --with-openssl --with-mhash --enable-pcntl --enable-sockets --with-ldap --with-ldap-sasl --with-xmlrpc --enable-zip --enable-soap --without-pear --with-zlib --enable-pdo --with-pdo-mysql

注意这里的 --with-mysql 如果在编译mysql的时候 没有指定-DINSTALL_LIBDIR=/usr/lib 那么这里需要写成mysql的安装路径
如果报 Don't know how to define struct flock on this system, set --enable-opcache=no 是由于没有找到mysql的库文件导致的
可以先运行ldconfig -v 命令 看看mysql的库文件有没有被加载进来 这个ldconfig就是用来加载动态链接库的
如果没有被加载进来  编辑 vi /etc/ld.so.conf.d/local.conf （这个local.conf 是随便起的 *.conf 的文件都会被加载）
在这个文件里加入你的库文件路径 例如 /usr/lib  保存退出后再次运行 ldconfig -v 注意不要重复引入 然后重新编译

make && make install

##php memcached扩展安装
 wget https://launchpad.net/libmemcached/1.0/1.0.18/+download/libmemcached-1.0.18.tar.gz
 tar -zxvf libmemcached-1.0.18.tar.gz
 cd libmemcached-1.0.18
 ./configure --with-memcached --prefix=/usr/local/libmemcached
 make && make install

 wget http://pecl.php.net/get/memcached-2.2.0.tgz
 tar -zxvf memcached-2.2.0.tgz
 cd memcached-2.2.0
 /usr/local/php/bin/phpize
 ./configure --enable-memcached --with-php-config=/usr/local/php/bin/php-config --with-libmemcached-dir=/usr/local/libmemcached
 make && make install






