---
title: JVM系列（一）java 类文件结构 
date: 2017-08-20 13:38:10
tags: 
- java
- jvm
category:
- java
---

## 什么是class文件
java 类文件即经过javac 编译后的文件，它的作用可以理解为用一种约定好的规则去描述一个类，jvm在拿到这个class文件后因为有了大家都知道的规则它就能解读理解这个类，比如这个类有哪些字段、有哪些方法、继承关系等等信息，就好像在阅读一本说明书一样。

因为这本说明书是给机器看的所以没有必要像人类的说明书一样有那么多空格换行注释，类文件是以字节为单位一个个紧密排列的二进制文件。

## class文件的结构

class文件中特定位置字节所表示的含义是固定的，比如固定的前4个字节表示魔数，5~6字节表示次版本号，文件中只包含两种基本结构: `无符号数` 和 `表`，无符号数以u1、u2、u3、u4 表示1个字节、2个字节、3个字节、4个字节，而表就是由多个无符号数或者子表构成的一种复合结构类似结构体，一个表的前面紧跟的肯定是一个u2用来表示这个表的长度。

下面是类文件的具体结构，不急着看懂每一项的含义，只要先有个直观的感受，后面我们会结合一个实例去说，想象这个文件是一个个排列好的格子，有的大格子（表）里面可能还套着n个小的格子，这些一个个排在一起的格子就构成了我们的类文件。

| 类型 | 名称 | 数量 |说明|
| --- | --- | --- |---|
|u4	|magic	|1|魔数，用于表示这个文件是个class文件|
|u2	|minor_version	|1|次版本号|
|u2	|major_version	|1|主板本号|
|u2	|constant_pool_count	|1|常量池长度|
|cp_info	|constant_pool	|constant_pool_count - 1|常量池表|
|u2	|access_flags	|1|访问标志|
|u2	|this_class	|1|当前类在常量池中的索引位置|
|u2	|super_class	|1|父类在常量池中的索引位置|
|u2	|interfaces_count	|1|接口的数量|
|u2	|interfaces|	interfaces_count|接口表|
|u2	|fields_count	|1|字段的数量|
|field_info	|fields	|fields_count|字段表|
|u2	|methods_count	|1|方法的数量|
|method_info|	methods	|methods_count|方法表|
|u2	|attribute_count|	1|属性的数量|
|attribute_info	|attributes	|attributes_count|属性表|

前三个格子最简单分别表示魔数、次版本号、主版本号都是用无符号数去表示，这三个一共占了8个字节（4+2+2），这8个字节结束之后就碰到了第一个表结构也是最基础的一个表`常量池表`，首先是一个u2字段表示常量池有多少项，紧接着就开始了这个表中的真正内容。

简单的说常量池就是把类中的各种元素像积木一样拆成一堆小元素，这些小元素主要分为两种：`字面量` 和 `符号引用`，字面量最好理解比如我们类中写了一个整数32769那这个数就会出现在常量池中，符号引用呢则是用来指代一个类或者一个方法或者一个字段，具体的内容我们后续再说。再提醒一点注意的是，我们这里说的是class文件中的常量池，不是运行时常量池，不要搞混。

常量池表结束后是访问标志字段，这个字段是表示限定于整个类上的一些标识，比如这是个类还是接口、是不是public的、是不是abstract的等等。

再往后是this_class 和 super_class 字段 这两个字段，用于表示当前类的全类名 和 父类的全类名，但是你发现他们只是一个u2那怎么表示一个可能路径很深的类呢，其实这里存的只是常量池中的一个索引值，细节后续再说。

继续往后又是一个表结构接口表，表示这个类实现了哪些接口。

接着是字段表，用于描述这个类有哪些字段。

接着是方法表，用于描述这个类有哪些方法。

最后是属性表，这表相当类上的一些附加信息，我们可以把一些扩展字段放到这里面，比如一个包含内部类的类，就会在这个地方存放内部类的列表。

整体的结构就介绍完了，总结一下大致的结构顺序

> 魔数 版本号  常量池  访问标志  当前类索引  父类索引 接口表 字段表 方法表 属性表

## 实例

下面我们就利用javap来直观的感受一下的类结构

java类

```
public class TestClass {
    public int anInt=888;

    public int otherInt=32769;

    public String anString = "an string";

    public int anFunction(int anPam){
        Integer a = new Integer(666);

        return a.intValue() + anPam + anInt;
    }
}

```
 

先用javac 编译这个类形成class文件，然后用javap查看编译后的类文件。

```
javac TestClass.java 
javap -v -p TestClass > TestClass.code
```

先看下编译后的TestClass.class文件长什么样

![](/media/15046186369105.jpg)

和前面介绍的一样，开始的一个u4是固定的魔数CAFEBABE， 紧跟的两个u2是次版本号主板本号，再后面是一个u2 0026转成10进制是38 表示常量池的长度是38，再往后就开始常量池、访问标志、类索引等等一系列的东西。

直接看二进制文件太反人类，我们用javap来解析出更易读结果（或者说反编译），输出的结果如下。

```
Classfile /Users/leeco/code/java-gc/src/TestClass.class
  MD5 checksum 28eefbbb8c29faedb096e39a37a1db17
  Compiled from "TestClass.java"
public class TestClass
  minor version: 0
  major version: 52
  flags: ACC_PUBLIC, ACC_SUPER
Constant pool:
   #1 = Methodref          #11.#25        // java/lang/Object."<init>":()V
   #2 = Fieldref           #10.#26        // TestClass.anInt:I
   #3 = Integer            32769
   #4 = Fieldref           #10.#27        // TestClass.otherInt:I
   #5 = String             #28            // an string
   #6 = Fieldref           #10.#29        // TestClass.anString:Ljava/lang/String;
   #7 = Class              #30            // java/lang/Integer
   #8 = Methodref          #7.#31         // java/lang/Integer."<init>":(I)V
   #9 = Methodref          #7.#32         // java/lang/Integer.intValue:()I
  #10 = Class              #33            // TestClass
  #11 = Class              #34            // java/lang/Object
  #12 = Utf8               anInt
  #13 = Utf8               I
  #14 = Utf8               otherInt
  #15 = Utf8               anString
  #16 = Utf8               Ljava/lang/String;
  #17 = Utf8               <init>
  #18 = Utf8               ()V
  #19 = Utf8               Code
  #20 = Utf8               LineNumberTable
  #21 = Utf8               anFunction
  #22 = Utf8               (I)I
  #23 = Utf8               SourceFile
  #24 = Utf8               TestClass.java
  #25 = NameAndType        #17:#18        // "<init>":()V
  #26 = NameAndType        #12:#13        // anInt:I
  #27 = NameAndType        #14:#13        // otherInt:I
  #28 = Utf8               an string
  #29 = NameAndType        #15:#16        // anString:Ljava/lang/String;
  #30 = Utf8               java/lang/Integer
  #31 = NameAndType        #17:#35        // "<init>":(I)V
  #32 = NameAndType        #36:#37        // intValue:()I
  #33 = Utf8               TestClass
  #34 = Utf8               java/lang/Object
  #35 = Utf8               (I)V
  #36 = Utf8               intValue
  #37 = Utf8               ()I
{
  public int anInt;
    descriptor: I
    flags: ACC_PUBLIC

  public int otherInt;
    descriptor: I
    flags: ACC_PUBLIC

  public java.lang.String anString;
    descriptor: Ljava/lang/String;
    flags: ACC_PUBLIC

  public TestClass();
    descriptor: ()V
    flags: ACC_PUBLIC
    Code:
      stack=2, locals=1, args_size=1
         0: aload_0
         1: invokespecial #1                  // Method java/lang/Object."<init>":()V
         4: aload_0
         5: sipush        888
         8: putfield      #2                  // Field anInt:I
        11: aload_0
        12: ldc           #3                  // int 32769
        14: putfield      #4                  // Field otherInt:I
        17: aload_0
        18: ldc           #5                  // String an string
        20: putfield      #6                  // Field anString:Ljava/lang/String;
        23: return
      LineNumberTable:
        line 1: 0
        line 2: 4
        line 4: 11
        line 6: 17

  public int anFunction(int);
    descriptor: (I)I
    flags: ACC_PUBLIC
    Code:
      stack=3, locals=3, args_size=2
         0: new           #7                  // class java/lang/Integer
         3: dup
         4: sipush        666
         7: invokespecial #8                  // Method java/lang/Integer."<init>":(I)V
        10: astore_2
        11: aload_2
        12: invokevirtual #9                  // Method java/lang/Integer.intValue:()I
        15: iload_1
        16: iadd
        17: aload_0
        18: getfield      #2                  // Field anInt:I
        21: iadd
        22: ireturn
      LineNumberTable:
        line 9: 0
        line 11: 11
}
SourceFile: "TestClass.java"

```

别被吓到，我们不用去深究这个输出的细节，先大体的知道每一块表示的意思即可。

首先最重要的一块 Constant pool， 常量池，我们简单的看下它，这个常量池的第一项（#1 = Methodref   #11.#25）是个方法的符号引用，一看就明白代表Object 的无参数默认实例化方法，至于那串奇怪的java/lang/Object."<init>":()V 是什么我们后面再说现在大概知道他是指代一个方法的引用即可，第二项呢（#2 = Fieldref    #10.#26）是一个字段的符号引用对应我们代码中的anInt这个变量，第三项呢就是一个字面量32769，第四项呢是一个字符串的符号引用可以看到他是引用的第28项，我们看下第28项呢是一个Utf8类型的字面量内容正是我们定义的`an string`，好了其他的我们先不去深究 ，你能感觉出来编译器把我们类中的关键信息都拆解了出来放到了常量池，你可能会说这样啰嗦的定义是搞甚?我的理解一是为了节省空间二是有个统筹管理的地方，把一些共用的东西都尽量抽出来，其实有点像我们自己在写程序中定义的一些常量，我们不也是把程序中多次用到的一些值抽出来定义成一个常量。

另一块，属性表，不多说，能看出来我们定义的三个属性。

```
public int anInt;
    descriptor: I
    flags: ACC_PUBLIC

  public int otherInt;
    descriptor: I
    flags: ACC_PUBLIC

  public java.lang.String anString;
    descriptor: Ljava/lang/String;
    flags: ACC_PUBLIC
```

最后一大块，方法表，可以看到两个方法，一个是自动生成无参数数，一个是我们定义的anFunction方法， 方法里面有个很重要的属性`Code`，没错它就表示了我们在方法中书写的代码逻辑，里面的每一行都是一个字节码指令。
对照我们定义的方法，我们简单看下这些字节码是啥。

```
public int anFunction(int anPam){
        Integer a = new Integer(666);
        return a.intValue() + anPam + anInt;
}

```

```
 0: new           #7    // class java/lang/Integer
 3: dup
 4: sipush        666
 7: invokespecial #8    // Method java/lang/Integer."<init>":(I)V
10: astore_2
11: aload_2
12: invokevirtual #9    // Method java/lang/Integer.intValue:()I
15: iload_1
16: iadd
17: aload_0
18: getfield      #2    // Field anInt:I
21: iadd
22: ireturn

```

具体的指令不用死记硬背用到时查手册，从这个例子中弄明白什么是基于栈的操作才是最重要的，简单说在调用一个方法之前必须为这个方法在操作数栈中准备好它需要的一些值，这里会涉及到局部变量表，操作数栈，堆这些概念不理解也没关系，我们下一节细说。

1. new #7  ，new 是指令 #7是参数表示常量池中的第7个元素，我们看到#7里存的是java/lang/Integer这个类的符号引用，合起来的意思就是new 一个Integer对象出来到堆里并把这个new出来的实例的引用压入操作数栈顶（操作数栈顶是什么我们后面将，这里先简单知道字节码的操作是基于栈的），这句执行完之后内存中就已经有个新的Integer对象，但是注意这时候对象中字段的值还都是0值，因为还没调用构造函数进行对象的初始化。

2. dup 复制刚才我们new出来的对象引用，并压入操作数栈，此时我们的操作数栈有了两个对那个Integer实例的引用
3. sipush 666, 把666这个字面量压入到栈顶，此时我们的操作数栈里面有个三个值
4. invokespecial #8 调用Integer类的构造函数，他会消耗掉我们栈中的两个值，一个是字面量666，一个是新对象的符号引用，方法调用完会把方法的返回值压入到我们当前的操作数栈帧，这个时候我们的栈帧中还剩一个符号引用指向那个Integer实例,说到方法调用这里提一句，即使是调用自己所在类的方法，也是要通过符号引用的。
5. astore_2 将操作数栈中的第一个元素弹出，并存入到局部变量表的第2个位置，即存入 a 这个变量中，（第0个位置是当前对象，第1个位置是函数参数anPam）,到这一步就完成了Integer a = new Integer(666); 这个语句的全部操作。到此时我们的栈里面就空了。
6. aload_2 还记得前面说过调用一个函数前必须在操作数栈中为它准备好要用到的值，这一句的作用就是把第二个局部变量压入操作数栈
7. invokevirtual #9  开始调用Integer.intValue方法，并把返回的值压入栈顶，此时栈中又有了一个字面量 666
8. iload_1 把局部变量表的第一个值压入栈，此时栈中有了两个值一个666 另一个是参数anPam上存的值
9. iadd 弹出操作数栈中的两个值进行加法运算，把结果压回栈
10. aload_0 把这个方法所在的实例（this）的引用压入操作数栈
11. getfield      #2，弹出栈顶那个实例，取它一个字段的值，并把这个值压入操作数栈，具体取哪个字段有参数#2知道，我们查一下常量表发现#代表的是anInt 这个字段的引用。这个时候我们的操作数栈中有两个值一个是前面iadd的计算结果  一个是这里刚拿到的字段anInt的值
12. iadd 再次执行加法运算，并把结果压回操作数栈，此时操作数栈中存放的值 就是a.intValue() + anPam + anInt 计算后的结果
13. ireturn 从当前方法返回栈顶的值

我们可以看到简单的一个new操作其实是对应虚拟机中好几部的操作，包括分配内存，初始化内存，赋值给变量等等，当然这中间可能还隐藏了类的加载，符号引用解析成直接地址等等，这些我们后面再细说。

最后再补充一下我们说的这些字节码，此刻还都是躺在静态的文件里，还没被真正的执行，上面说的那一串东西都是描述假如这段代码被执行了是个什么效果。

##总结
到此应该对class文件是什么，有什么作用有了一个直观的感受，这个文件中的一些细节比如字段表的结构是怎么样的，函数的异常是定义在哪的，我们可以在用到的时候再去细究，下一节我们开始讨论之前提到的堆、操作数栈、局部变量表这些东西。




