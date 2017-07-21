title: "Yii 学习笔记: 安装"
date: 2015-05-27 10:40:33
tags: 
- Yii
category:
- php
---

#参考
[Yii中文文档](http://www.yiichina.com/doc/guide/2.0/start-installation)
[配置composer](http://xrong.net/2014/09/20/%E5%9B%BD%E5%86%85%E4%BD%BF%E7%94%A8composer%E7%9A%84%E6%AD%A3%E7%A1%AE%E5%A7%BF%E5%8A%BF/)

composer是php的包管理工具 yii的项目也可通过它去初始化 由于要获取github上的项目 在国内不配置代理或镜像很卡 

为了愉快的敲代码 首先就先把这个composer干掉
我用的是window下的git bash 这个里面集成了很多linux下的shell命令 简单方便
```
$ composer config -l -g | grep home
[home] C:/Users/Administrator/AppData/Roaming/Composer
```
-l是list的缩写 -g是global的缩写 即列出全局的配置信息 然后我们筛选得到home这一项 这个就是composer的工作目录 配置文件 以及全局的安装都会存在这里

```
$ composer config -e -g 
```
这条命令将会用户编辑器打开全局的配置文件 
将配置文件修改为
```
{
    
	"config": {
	},
	"repositories": [
        	{"type": "composer", "url": "http://pkg.phpcomposer.com/repo/packagist/"},
        	{"packagist": false}
    	]

}

```

开始安装第一个全局的composer插件 composer-asset-plugin 这个插件用于管理一些前台包 如css js  框架等

```
$ composer global require "fxp/composer-asset-plugin:~1.0.0"
```
~号的意思是 大于1.0版本但小于2.0版本的软件包
