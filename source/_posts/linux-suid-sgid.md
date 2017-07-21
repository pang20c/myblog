title: linux 下的suid和sgid的作用
date: 2014-11-20 11:19:30
tags:
- linux
category:
- linux
---

这几天集中扫除自己不清楚的一些linux基础概念，今天搞一下suid和sgid

这俩东西是干嘛用的呢 它主要用在一些可执行文件上 简单说就是让一个用户在执行这个程序的时候拥有这个程序创建者的权限

关于这个东西最常举的一个例子就是passwd这个命令 

我们知道一些敏感文件的读写权限是只有root才可以的 但是有时候普通用户又需要间接的去用这个文件
比如passwd这个命令 他的执行文件是在 /usr/bin/passwd 这个执行文件是一个程序而这个程序需要操作的数据来自 /etc/passwd 等普通用户没权限的文件
```bash
[root@localhost ~]# ll /usr/bin/passwd /etc/passwd
-rw-r--r--. 1 root root  1584 Aug  1 00:33 /etc/passwd
-rwsr-xr-x. 1 root root 30768 Feb 22  2012 /usr/bin/passwd
```
虽然/usr/bin/passwd 每个用户都能去执行它 但是光有程序没数据是不行的啊 怎么办？
这时候suid和sgid就为了解决这个问题出现了
我们注意到 /usr/bin/passwd 的用户执行位是个s而不是常见x 这个就说明这个文件设定了suid属性
我以前一直错误的以为suid是一个具体用户的id其实不是的 它只是一个标志位 标志这个文件是不是设置过suid

具体的说当一个用户去执行passwd这个命令的时候，这个命令的是以创建者root的身份启动的 而不是这个命令的执行者

我们做个试验 打开俩终端 一个登陆root 一个登陆一个普通用户kuff
在kuff的终端去执行passwd命令 
```bash
login as: kuff
kuff@192.168.0.183's password:
[kuff@localhost ~]$ passwd
Changing password for user kuff.
Changing password for kuff.
(current) UNIX password:
```
在root的终端查看进程
```bash
[root@localhost ~]# ps -ef | grep passwd
root      8677  8654  0 19:50 pts/1    00:00:00 passwd
root      8680  1713  0 19:50 pts/0    00:00:00 grep passwd
```

看到了吧 在1终端的上的passwd命令是以root身份执行的 
至于sgid 理解了suid不用说应该也能理解了

说了这么多怎么给一个文件设置sgid？
chmod u+s a.txt #设置sgid
chmod u-s a.txt #取消设置sgid

当然所有设置了sgid的程序都是存在潜在风险的 因为程序如果有漏洞 别人就能拿这个程序直接执行root才能执行的东西


