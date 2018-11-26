---
title: 'Python编程:从入门到实践-学习笔记1'
date: 2018-11-22 21:19:37
tags: python
categories: 后端
---

Python编程：从入门到实践-学习笔记系列第一篇

* 列表
* if语句
* 字典
* 用户输入和while循环
* 函数
* 类

------

<!-- more -->

## 列表

#### 列表简介

1. 索引从0而不是从1开始
2. 添加元素：append()末尾新增，insert()在指定位置新增
3. 删除元素：del语句删除，pop()删除任何位置的元素，remove()删除值
4. 组织列表：sort()永久性排序，sorted()临时排序，reverse()反转列表



#### 操作列表

1. 避免缩进错误，Python根据缩进来判断代码行和前一个代码行的关系

2. for语句末尾有冒号

3. 使用range()函数创建数字列表，可使用list()将range()结果转为列表

4. 列表解析：将for循环和创建新元素的代码合并成一行

   ```python
   squares = [value**2 for value in range(1, 11)]
   ```

5. 切片：players[0:3]、players[:4]、players[2:]、players[-3:]
6. 复制列表，同时省略起始索引和终止索引（[ : ]）
7. 元组是不可变的列表，用圆括号表示
8. 不能修改元组的元素，但可以给存储元素的变量赋值

## if语句

1. 使用and和or检查多个条件
2. 使用in判断特定的值是否已包含在列表中，not in不包含在列表中
3. if-elif-else结构
4. if语句判断列表时，列表至少包含一个元素返回True，列表为空返回False

## 字典

1. 使用del语句删除键-值对
2. `for k, v in user.items()`、 `for k in user.keys()`、`for v in user.values()`

## 用户输入和while循环

1. input()等待用户输入、int()将输入视为数值
2. 要在遍历列表的同时对其修改，可使用while循环

## 函数

1. 使用def来进行函数定义
2. 文档字符串：3个引号
3. 位置实参：实参的顺序和形参的顺序相同；关键字实参：传递给函数的名称-值对；可给每个形参指定默认值
4. 任意数量的实参：*keys；任意数量的关键字实参：**keys（创建一个keys的空字典）
5. 模块使扩展名为.py的文件；导入模块import module，调用被导入模块的函数使用module.function()
6. 导入特定的函数：`from module import function1, function2, function3`
7. 使用as给函数指定别名：`from module import function1 as fn`；使用as给模块指定别名：`import module as  mn`
8. 导入模块中所有函数：`from module import *`（尽量不要，可能重名）

## 类

1. 方法`__init__()`，通过类创建新实例，会自动运行；形参self必不可少，必须位于最前面，创建实例时，将自动传入实参self：

   ```python
   def __init__(self, name, age)
   ```

2. 变量前有self作为前缀，这样的变量称为属性
3. 修改属性的值：直接修改属性的值；通过方法修改属性的值；通过方法对属性的值进行递增
4. 继承：class SubClass(SuperClass)，在子类的`__init__()`中调用`super().__init__()`进行关联