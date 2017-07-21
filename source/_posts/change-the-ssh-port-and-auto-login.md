title: putty通过密钥自动登陆 关闭密码登陆
date: 2014-09-24 11:45:04
tags:
- ssh
- linux
- security
category:
- linux
---
自己的服务器总是被别人扫描 看着心里就不爽 于是决定改ssh的登陆关口关掉密码登陆

#修改ssh登陆端口
这个简单修改sshd的配置文件就OK了
打开 /etc/ssh/sshd_config
修改里面的Port

```bash
Port 2014
#ListenAddress 0.0.0.0
#ListenAddress ::
```
注意的是如果开着防火墙别忘了在防火墙上新开个端口

```bash
#加入新的端口
iptables -A INPUT -p tcp -m tcp --dport 2014 -j ACCEPT
#找到原来22端口的那条id 比如是1
iptables -nL --line-nunmber
#删除掉他
iptables -D INPUT 1
service iptables save
```

修改完重启下sshd服务

# 用密钥登陆服务器
大体的原理很简单就是一个非对称加密解密的过程
生成两个密钥（这里我用的[PuTTYgen](http://www.chiark.greenend.org.uk/~sgtatham/putty/download.html)）一个公钥 一个私钥
公钥放在服务器上 私钥放在你自己的电脑上 就OK了

我windows电脑上用的PuTTY 下面说用这个软件的免密码登陆流程
具体流程：

*	用PuTTYgen生成一对密钥,注意那个文字提示要鼠标不停的在那动才能生成(- -!)
![生成密钥](/img/putty_1.jpg)
生成完了之后私钥保存到你本地一个地方 他会提示你是否确认没设密码就存 我要无密码登陆那就不设了
公钥呢就没必要存了 那个进度条下面的那段文字 复制一下就行了 一会粘到服务器里去

*	到你服务器上去在用户目录下新建一个.ssh目录 在这个目录下新建一个 authorized_keys文件（当然也不一定叫这个可以在上面说到的sshd_config里去修改这个文件名）
编辑这个文件把刚才的公钥字符串粘进去 其实这个文件是可以有多个公钥的 一行一个就行了

*	回到你本地putty上配置
编辑一条记录,选择你刚才生成的私钥，别忘了保存
![修改密钥文件地址](/img/putty_2.jpg)
好了现在去登陆吧
会提示你输入用户名 输入完用户名不用密码就能登陆进去了
如果你连用户名也不想输入 那编辑putty的默认用户名就行了
![修改默认用户名](/img/putty_3.jpg)

#关闭密码登陆
还是编辑/etc/ssh/sshd_config
找到最下面的
```bash
PasswordAuthentication no
#默认是yes改成no就行了
```
重启sshd
```
service sshd restart
```



