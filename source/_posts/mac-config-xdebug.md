---
title: mac 环境配置xdebug 到phpstorm
date: 2017-08-04 18:03:06
tags: 
- xdebug
category:
- php
---

本文的前置条件是使用brew安装php环境，其他方式请避让

## 安装xdebug扩展

`brew install php56-xdebug` 安装完成后查看`brew info php56-xdebug` 查看xdebug配置文件路径 我的路径是`/usr/local/etc/php/5.6/conf.d/ext-xdebug.ini`

修改这个配置文件 增加远程调用的支持

```
xdebug.remote_enable = On
xdebug.remote_port = 9004
```
**注意**这里的端口 不能和php-fpm的重复 所以改为9000之外的其他端口

重启php-fpm `brew services restart php56`

运行phpinfo查看 xdebug是否配置正确

## 配置 phpstorm
Phpstrom--> Preferences.. 找到php选项，配置cli地址
![](media/15018418358978.jpg)

![](media/15018418757327.jpg)

再找到php文件夹下的debug配置xdebug的端口，注意要和ini文件中配置的一致才能远程调试

![](media/15018420640780.jpg)

## 创建Debug配置
可以远程调试的前提是 已经配置好了nginx或其他服务器 可以用本地ip访问到你的php页面

增加调试配置
![](media/15018442278043.jpg)

增加一个web application 配置，serverc处点击更多，配置本地的服务器地址和端口
![](media/15018443219352.jpg)

至此配置完毕 可以点击绿臭虫图标开始断点调试








