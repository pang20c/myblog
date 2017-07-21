title: linux 用户组
date: 2014-11-19 22:39:59
tags: 
- linux
category:
- linux
---
发现自己对linux中用户组的这个概念还是比较模糊的今天又翻了翻书总结记录一下

当我们添加一个用户的时候如果没有给这个用户指定一个用户组 那么系统会创建一个和用户名相同的用户组
我们用 useradd kuff 添加一个测试用户
再用 groupadd testgroup 添加一个测试用户组
把刚才的测试用户加入到测试组 gpasswd -a kuff testgroup
这时候再去查看/etc/group 中的内容发现多了两个用户组
```bash
$ tail -2 /etc/group
kuff:x:501:
testgroup:x:502:kuff
```

既然叫组，那一个用户自然可以属于多个组，怎么看自己属于那些组呢？
```bash
$ su - kuff #切换到kuff用户
$ groups
kuff testgroup
```
可以看到kuff属于两个组 这时候如果去创建一个文件 文件的组是属于kuff组的
```bash
$ touch 1.txt
$ ll
total 0
-rw-rw-r-- 1 kuff kuff 0 Nov 19 23:00 1.txt
```

那如果我想切换到我的另外一个组呢？newgrp
```bash
$ newgrp testgroup
$ groups
testgroup kuff
```
发现变化了吧？testgroup挪到前面去了 此时再去创建文件 新文件的属组变成了 testgroup

这里涉及到俩概念 __初始用户组__  和 __有效用户组__
* 	初始用户组呢就是创建用户的时候指定给它的用户组（没指定就是用户名同名用户组） 初始用户组并不会在/etc/group的成员中看到因为它已经记录到用户身份信息中了
	举个例子我们在添加一个用户kuff2 这次指定他的组为前面的kuff

```bash
$ useradd -gkuff kuff2
$ grep kuff /etc/group /etc/passwd
/etc/group:kuff:x:501:
/etc/group:testgroup:x:502:kuff
/etc/passwd:kuff:x:501:501::/home/kuff:/bin/bash
/etc/passwd:kuff2:x:502:501::/home/kuff2:/bin/bash
```

我们看到虽然kuff2用户属于kuff用户组但是 在group文件的最后一项成员中并没有它

*	有效用户组呢就是代表用户当前使用的组 也就是groups排在前面的那个用户组 当用户去创建新文件的时候就会根据这个用户组去决定文件属于哪个组
用户刚登陆时有效用户组就是初始用户组 

当然在判断用户读写权限的时候是不区别是不是有效用户组的 只要他在那个组里就可以读写