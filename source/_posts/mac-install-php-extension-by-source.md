---
title: mac 下通过源码安装php扩展
date: 2017-08-03 15:40:25
tags:
- php
category:
- php
---
# 安装源码包中存在的扩展
以openssl为例进行安装 其他扩展类似

## 下载源码
- 下载`git clone git@github.com:php/php-src.git`
- 执行`git tag | grep 5.6.0`找到需要的源码版本例如`php-5.6.0`，并切到此tag下`git checkout php-5.6.0`

## 执行phpize

进入到扩展目录`ext/openssl`执行`phpize` 提示找不到config.m4 这个文件，解决办法`mv config0.m4 config.m4` 后，继续执行phpize 通过

## 执行configure

`./configure --with-php-config=/usr/local/bin/php-config` 报错提示找不到openssl头文件 `Cannot find OpenSSL's <evp.h>`,

解决方案`./configure --with-php-config=/usr/local/bin/php-config --with-openssl=/usr/local/opt/openssl`

分析过程：

1. 首先尝试安装openssl，`brew install openssl` 安装后configure依然不能通过
2. brew info openssl 查看安装信息，发现文末有这么一句提示![](media/15017479570718.jpg)
3. 尝试指定安装目录`./configure --with-php-config=/usr/local/bin/php-config --with-openssl=/usr/local/opt/openssl` 成功

##编译安装so文件
`make && make install`
![](media/15017484579095.jpg)
用`php --info | grep php.ini`找到你的`php.ini`文件
在配置文件中添加`extension=openssl.so`

`php -m|grep openssl`确认扩展已经安装
如果是php-fpm 重启php-fpm


# 补充-如何用brew 安装php
官方的brew库里没有php需要先扩展brew的安装库
`brew tap homebrew/php`
然后执行 `brew install --without-apache --with-fpm --with-mysql php56`
把新安装的php加入到系统路径
`echo 'export PATH="$(brew --prefix homebrew/php/php56)/bin:$PATH"' >> ~/.zshrc` 注意我用zsh所有是.zshrc







