---
title: shell 中的查找替换
date: 2017-07-26 15:56:39
tags:
- shell
- sed
category:
- shell

---

* 用sed实现多文本内容替换


```
sed -i 's/abc/ABC/g' `grep -rl 'abc' /usr/local/`
```
其中`grep -l`的作用是只显示文件路径 不显示其他内容 方便传递给sed

