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