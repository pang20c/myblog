title: window下查看 apache 配置文件错误
date: 2015-05-07 15:20:44
tags:
- apache
category:
- apache
---

在Windows下修改Apache配置文件 导致服务没法启动了 但是从错误日志里却看不到错误信息 后百度到是可以从dos启动Apache然后看报错信息
具体命令如下
```
D:\wamp\bin\apache\apache2.4.9>bin\httpd.exe -w -k restart -n wampapache64
```
* -w 的作用是在控制台中输出错误信息
* -k restart 是一体的 表示重启apache
* -n 是Apache服务的名字这个是个要注意的地方 如果不填他会按默认的名字去启动 但是很多集成环境他已经改了服务的名字了 这个可以从计算机管理里的服务中看到真实的名字

更具体的说明 可以结合-h查看

终于看到报错信息了
```
AH00526: Syntax error on line 28 of D:/wamp/bin/apache/apache2.4.9/conf/extra/httpd-vhosts.conf:
RewriteBase: only valid in per-directory config files
Note the errors or messages above, and press the <ESC> key to exit.  0....
```

