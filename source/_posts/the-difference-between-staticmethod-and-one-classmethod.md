title: "python中的 @staticmethod 和 @classmethod"
date: 2014-09-26 18:31:39
tags:
- Python
category:
- Python
---
最近在学习python 看到关于类的介绍的时候一个让我迷惑的地方就是这两个装饰器 静态方法 和类方法

在php 我没有接触过类方法这个概念

经过一番百度谷歌 以下是我个人的见解如有错误 欢迎指正

> 其实classmethod和staticmethod的作用是极为相似的甚至可以说staticmethod的存才是有点多余的
- staticmethod完全可以用一个类外的方法替代，但是为了让代码看着更有组织纪律性让代码的逼格更高才会写静态方法
- classmethod更像是我们习惯的静态方法 但是他有个奇葩的地方 他可以用一个实例去调用 给你造成一种假象好像用实例去调用静态方法时他就是个实例方法其实他还是静态的全局的 - -！

再白话点说classmethod 说明这个方法还和类有着一丝联系（通过cls）而staticmethod则是一个工具方法 他是不应有状态的完全可以脱离开这个类

这种关系是很微妙的 你可以说我完全可以在staticmethod中通过类名去调用这个当前类的属性 实现和classmethod一样的功能啊，但是这是你硬写上去的 如果这个类被继承那么这种硬编码也就失效了

有点抽象 下面上具体代码说事
```python
class Annimal(object):
    debug = False
    weight = 0
    name = ''
    def __init__(self, weight,name):
        self.weight = weight
        self.name = name

    def eat(self, num): # 一个普通的类方法
        self.weight += num

    def outinfo(self):# 一个普通的类方法
        if(self.debug):
            print(self.__dict__)
        else:
            self.formatinfo(self.name, self.weight)

    @classmethod
    def isdebug(cls, debug=False): # 类方法
        cls.debug = debug

    @staticmethod
    def staticdebug(debug): # 静态方法
         Annimal.debug = debug

    @staticmethod
    def formatinfo(name, weight): # 静态方法
        print("the name is %s and weight is %d" % (name, weight))

class Cat(Annimal):
    pass

cat1 = Cat(10, 'whitecat')
cat2 = Cat(15, 'blackcat')
cat1.isdebug(True) #当然这里一般写成Cat.isdebug(True) 为了体现类方法特点我写成了用一个实例去调取
cat3 = Cat(20, 'flowercat') #是的花猫 - -！
cat1.outinfo()
cat2.outinfo()
cat3.outinfo()
print('-'*10, '用当前类调用类方法，会影响之后的所有实例')
Cat.isdebug(False)
cat1.eat(3)
cat2.eat(4)
cat1.outinfo()
cat2.outinfo()
print('-'*10, '用父类去调用类方法，不会影响子类')
Annimal.isdebug(True)
cat1.outinfo()
cat2.outinfo()
print('-'*10, '调用当前类的静态方法,哈哈这种硬编码的方式是不会作用到子类上的')
Cat.staticdebug(True)
cat1.outinfo()
cat2.outinfo()

```

这段程序的输出如下

```
{'weight': 10, 'name': 'whitecat'}
{'weight': 15, 'name': 'blackcat'}
{'weight': 20, 'name': 'flowercat'}
---------- 用当前类调用类方法，会影响之后的所有实例
the name is whitecat and weight is 13
the name is blackcat and weight is 19
---------- 用父类去调用类方法，不会影响子类
the name is whitecat and weight is 13
the name is blackcat and weight is 19
---------- 调用当前类的静态方法,哈哈这种硬编码的方式是不会作用到子类上的
the name is whitecat and weight is 13
the name is blackcat and weight is 19
```

