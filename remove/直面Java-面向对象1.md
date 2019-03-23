---
title: 直面Java-面向对象1
date: 2018-09-10 22:15:24
tags: Java
categories: 学习笔记
---

这个系列是关于Java基础的知识点，常用于面试。

* 面向对象的三大基本特征和五大基本原则
* 为什么说Java只有值传递
* Java的编译和反编译方法

------

<!-- more -->

## 面向对象的三大基本特征和五大基本原则

- 三大基本特征：封装、继承、多态
- 五大基本原则：单一职责原则（Single-Responsibility Principle）、开放封闭原则（Open-Closed Principle）、Liskov替换原则（Liskov-Substituion Principle）、依赖倒置原则（Dependency-Inversion Principle）和接口隔离原则（Interface-Segregation Principle）
  - 单一职责原则：一个类最好只做一件事，只有一个引起它的变化
  - 开放封闭原则：对扩展开放，对修改封闭
  - Liskov替换原则：子类必须能够替换其基类
  - 依赖倒置原则：依赖于抽象
  - 接口隔离原则：使用多个小的专门的接口，而不要使用一个大的总接口

## 为什么说Java只有值传递

#### 实参和形参

* 实参是调用有参方法的时候真正传递的内容
* 形参数用于接收实参内容的参数

#### 值传递和引用传递

* 值传递指调用函数时将实际参数**复制**一份传递到函数中，这样在函数中对参数进行修改，将不会影响到实际参数
* 引用传递指调用函数时将实际参数的地址**直接**传递到函数中，那么在函数中对参数进行的修改，将影响到实际参数

#### Java中的值传递

​	值传递和引用传递之间区别的重点是：

{% asset_img image1.webp 直面Java-面向对象1 %}

举一个例子：

```java
public static void main(String[] args) {
   ParamTest pt = new ParamTest();

   User hollis = new User();
   hollis.setName("Hollis");
   hollis.setGender("Male");
   pt.pass(hollis);
   System.out.println("print in main , user is " + hollis);
}

public void pass(User user) {
   user = new User();
   user.setName("hollischuang");
   user.setGender("Male");
   System.out.println("print in pass , user is " + user);
}
```

上面代码的输出结果：

```java
print in pass , user is User{name='hollischuang', gender='Male'}
print in main , user is User{name='Hollis', gender='Male'}
```

{% asset_img image2.webp 直面Java-面向对象1 %}

从上图分析我们可以得知，这里是把实际参数的引用地址**复制**了一份，传递给了形式参数。**所以，上面的参数其实是值传递，把实际参数引用的地址当做值传递给了形式参数。值传递和引用传递的区别并不是传递的内容，而是实参到底有没有复制一份给形参。**

**Java中其实还是值传递，对于对象参数，值的参数是对象的引用。**

## Java的编译和反编译方法

* 编译指的是从高级语言转换成低级语言，将用高级计算机语言所写作的源代码程序，翻译为低阶机器语言的程序的过程就是编译，负责这一过程的工具叫做编译器。
* 反编译指的是将已编译好的低阶语言还原到未编译的状态，就是找到程序语言的源代码。Java语言中的反编译一般指将class文件转换成java文件。

Java语言中负责编译的编译器是一个命令：javac。该工具可以将后缀为.java的源文件编译为后缀为.class的可以运行与Java虚拟机的字节码，再由JVM将这种class文件类型字节码转换成机器可以识别的机器码。

Java中反编译工具有多个，如`javap`，`jad`，`CRF`：

**javap**

`javap`是jdk自带的一个工具，可以对代码反编译，也可以查看java编译器生成的字节码。`javap`和其他两个工具最大的区别是它生成的并不是java代码，不像其他两个工具生成的代码那样容易理解。

源代码：

```java
package com.hyc.learn.demo.java;

public class SwitchDemoString {
    public static void main(String[] args) {
        String str = "world";
        switch (str) {
            case "hello":
                System.out.println("hello");
                break;
            case "world":
                System.out.println("world");
                break;
            default:
                break;
        }
    }
}
```

执行`javap -c SwitchDemoString.class`结果：

```java
Compiled from "SwitchDemoString.java"
public class com.hyc.learn.demo.java.SwitchDemoString {
  public com.hyc.learn.demo.java.SwitchDemoString();
    Code:
       0: aload_0
       1: invokespecial #1                  // Method java/lang/Object."<init>":()V
       4: return

  public static void main(java.lang.String[]);
    Code:
       0: ldc           #2                  // String world
       2: astore_1
       3: aload_1
       4: astore_2
       5: iconst_m1
       6: istore_3
       7: aload_2
       8: invokevirtual #3                  // Method java/lang/String.hashCode:()I
      11: lookupswitch  { // 2
              99162322: 36
             113318802: 50
               default: 61
          }
      36: aload_2
      37: ldc           #4                  // String hello
      39: invokevirtual #5                  // Method java/lang/String.equals:(Ljava/lang/Object;)Z
      42: ifeq          61
      45: iconst_0
      46: istore_3
      47: goto          61
      50: aload_2
      51: ldc           #2                  // String world
      53: invokevirtual #5                  // Method java/lang/String.equals:(Ljava/lang/Object;)Z
      56: ifeq          61
      59: iconst_1
      60: istore_3
      61: iload_3
      62: lookupswitch  { // 2
                     0: 88
                     1: 99
               default: 110
          }
      88: getstatic     #6                  // Field java/lang/System.out:Ljava/io/PrintStream;
      91: ldc           #4                  // String hello
      93: invokevirtual #7                  // Method java/io/PrintStream.println:(Ljava/lang/String;)V
      96: goto          110
      99: getstatic     #6                  // Field java/lang/System.out:Ljava/io/PrintStream;
     102: ldc           #2                  // String world
     104: invokevirtual #7                  // Method java/io/PrintStream.println:(Ljava/lang/String;)V
     107: goto          110
     110: return
}
```

从上面结果看，`javap`并没有将字节码反编译成java代码，而是生成了一种我们看得懂的字节码。其实`javap`生成的文件仍然是字节码。上面的字节码其实是把String转成hashcode，然后进行比较。

**jad**

jad是一个不错的反编译工具，只要下载一个可执行程序，就可以对class文件进行反编译。上面的源代码使用jad反编译结果如下：

运行命令：`jad switchDemoString.class`

```java
// Decompiled by Jad v1.5.8g. Copyright 2001 Pavel Kouznetsov.
// Jad home page: http://www.kpdus.com/jad.html
// Decompiler options: packimports(3) 
// Source File Name:   SwitchDemoString.java

package com.hyc.learn.demo.java;

import java.io.PrintStream;

public class SwitchDemoString
{

    public SwitchDemoString()
    {
    }

    public static void main(String args[])
    {
        String str = "world";
        String s = str;
        byte byte0 = -1;
        switch(s.hashCode())
        {
        case 99162322: 
            if(s.equals("hello"))
                byte0 = 0;
            break;

        case 113318802: 
            if(s.equals("world"))
                byte0 = 1;
            break;
        }
        switch(byte0)
        {
        case 0: // '\0'
            System.out.println("hello");
            break;

        case 1: // '\001'
            System.out.println("world");
            break;
        }
    }
}
```

 这个代码块就很清晰，很清楚的看到**字符串的switch就是通过`equals()`和`hashcode()`方法实现的**。

但是，jad已经很久没有更新了，官网提示不再更新了，不建议使用了。

**CRF**

jad虽然很好用，但是无奈已经停止更新，所以只能用新的工具去替代它，CFR是一个不错的选择，相比jad，它的语法会稍微复杂一些，但是也可以使用。

如，对上面的源代码进行反编译，输入以下命令：

```shell
java -jar cfr_0_131.jar SwitchDemoString.class  --decodestringswitch false
```

结果如下：

```java
/*
 * Decompiled with CFR 0_131.
 */
package com.hyc.learn.demo.java;

import java.io.PrintStream;

public class SwitchDemoString {
    public static void main(String[] args) {
        String str;
        String string = str = "world";
        int n = -1;
        switch (string.hashCode()) {
            case 99162322: {
                if (!string.equals("hello")) break;
                n = 0;
                break;
            }
            case 113318802: {
                if (!string.equals("world")) break;
                n = 1;
            }
        }
        switch (n) {
            case 0: {
                System.out.println("hello");
                break;
            }
            case 1: {
                System.out.println("world");
                break;
            }
        }
    }
}
```

通过上面的结果也可以看出字符串的switch是用`hashcode()`和`equal()`方法实现的。

和jad相比，CFR有很多参数，上面的代码用以下的命令输出结果就是不一样的：

```java
java -jar cfr_0_131.jar SwitchDemoString.class

/*
 * Decompiled with CFR 0_131.
 */
package com.hyc.learn.demo.java;

import java.io.PrintStream;

public class SwitchDemoString {
    public static void main(String[] args) {
        String str;
        switch (str = "world") {
            case "hello": {
                System.out.println("hello");
                break;
            }
            case "world": {
                System.out.println("world");
                break;
            }
        }
    }
}
```

参数`--decodestringswitch`表示对于switch支持string的原理进行解码。类似的还有`--decodeenumswitch`、`--decodefinally`、`--decodelambdas`。CFR还有很多其他的参数，可以使用`java -jar cfr_0_131.jar --help`进行了解

**如何防止反编译**

对于反编译来讲，其实和网络安全的防护，只能提高攻击者的成本而已，并不能彻底防治。

应对策略有以下几种：

* 隔离应用程序
  * 让用户接触不到Class文件
* 对Class文件进行加密
  * 提高破解难度
* 代码混淆
  * 将代码转化为功能上等价，但是难以阅读和理解的形式。

## 参考链接:

http://www.hollischuang.com/archives/220

https://mp.weixin.qq.com/s/F7Niaa7nD1tLApCEGKAj4A