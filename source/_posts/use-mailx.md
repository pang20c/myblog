title: 用mailx 在linux上发送邮件
date: 2014-09-23 16:09:00
tags:
- linux
- mail
category:
- linux
---
最近突然想把服务器上的日志 通过邮件发送出去 于是收集整理了一下关于在linux上发送邮件的知识

#理论基础

## MUA MTA MDA
[http://emo17emo17.blogspot.com/2009/03/blog-post_30.html](http://emo17emo17.blogspot.com/2009/03/blog-post_30.html)
[http://blog.sina.com.cn/s/blog_726618d901011n8s.html](http://blog.sina.com.cn/s/blog_726618d901011n8s.html)

网上关于邮件发送流程分析的文章很多 找了两篇讲解的比较好的
最主要的是理解MUA MTA MDA 三个概念 前两个比较好理解 一个是客户端一个是服务器 但是这个MDA看了好几篇文章才算清楚了点

其实邮件服务器的结构很类似我们常见的http服务器 

MTA 就像是nginx 他接收来自客户端的请求 但是MTA他只是接收和发送功能 他是不处理具体的邮件逻辑的 他只是网络部分

MDA 呢就像是nginx后面cgi程序 他才是真正处理邮件逻辑的程序 例如过滤垃圾邮件啊 自动回复什么的

## SMTP POP3/IMAP 

邮件协议
简单的说SMTP 是发邮件用的 POP3/IMAP 是读邮件用的

至于POP3和IMAP的区别 见网易的一个[解释](http://help.163.com/10/0203/13/5UJONJ4I00753VB8.html?servCode=6010237) 

#发送邮件
我们不是要搭建一个邮件服务器 我们只是要在linux上发送邮件 
也就是说我们要扮演的是个MUA的角色 至于怎么搭建自己的邮件服务器 以后学会了再说现在还是看着挺晕的
用到的是系统自带的mailx软件
```bash
rpm -qa | grep mailx
libreport-plugin-mailx-2.0.9-19.el6.centos.x86_64
mailx-12.4-7.el6.x86_64
```

他的配置文件在
```
/etc/mail.rc
```
编辑这个文件
加上咱的smtp服务器配置信息
```bash
set from=pengling@163.com
set smtp=smtp.163.com
set smtp-auth-user=pengling
set smtp-auth-password=111111
set smtp-auth=login

```

好了配置完了 
发个邮件试试
```bash
echo hello word | mailx -v -s '测试邮件' 461921766@qq.com
```

OK 
