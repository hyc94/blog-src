---
title: JavaScript全栈教程-学习笔记1
date: 2018-11-20 21:51:24
tags: JavaScript
categories: 前端
---

JavaScript全栈教程-学习笔记系列第一篇

* 快速入门
* 函数

------

<!-- more -->

## 快速入门

1. 字符串：模板字符串（ES6新增）
2. 数组：
   - length赋新值会导致Array大小的变化
   - `slice()`、`unshift()`、`shift()`、`splice()`万能方法
3. 对象：
   - `delete  object.prop`删除属性
   - `in`操作符判断是否有一属性（可能是继承得到的）
   - `hasOwnProperty()`是自身拥有的
4. Map和Set是ES6标准新增的数据类型
5. iterable
   - 具有iterable类型的集合可以通过`for ... of`循环来遍历
   - `for ... of`循环和`for ... in`循环的区别：`for ... in`循环由于历史遗留问题，它遍历的实际上是对象的属性名称;`for ... of`只循环集合本身的元素
   - `iterable`内置的`forEach`方法

## **函数**

1. 函数的定义和作用
   - 关键字`arguments`，它只在函数内部使用，指向当前函数的调用者传入的参数。`arguments`最常用于判断参数的个数
   - ES6标准引入了获得额外参数的`rest`参数，`rest`参数只能写在最后，前面用...标识
   - JavaScript引擎会在行末自动添加分号的机制，如果return语句拆成两行，可能会出错
2. 变量作用域与解构赋值
   - 变量提升
   - 全局作用域：JavaScript默认有一个全局对象`window`，`alert()`函数其实也是`window`的一个变量
   - 名字空间
   - 局部作用域：变量作用域实际上是函数内部，为了解决块级作用域，ES6引入了新的关键字`let`
   - 常量：ES6引入了关键字`const`来定义常量，`const`与`let`都具有块级作用域
   - 解构赋值，可以同时对一组变量进行赋值
3. 方法
   - 方法内部的`this`可能指向`undefined`，通过一个`that`变量首先捕获`this`
   - 要指定函数的`this`指向哪个对象，可以用函数本身的`apply()`。`call()`与`apply()`类似，`apply()`把参数打包`Array`再传入，`call()`把参数顺序传入
4. 高阶函数：`map()`、`reduce()`、`filter()`、`sort()`
5. 闭包（函数作为返回值）
   - 返回函数不要引用任何循环变量，或者后续会发生变化的变量
   - 如果一定要引用循环变量，可以再创建一个函数，用该函数的参数绑定循环变量当前的值
   - 闭包作用：封装私有变量、把多参数的函数变成单参数的函数
6. 箭头函数（ES6标准）
   - 箭头函数和匿名函数的区别：箭头函数内部的`this`是词法作用域，由上下文确定。
7. generator
   - generator由`function*`定义，并且除了`return`语句，还可以用`yield`多次返回
   - 调用generator对象有两个方法，一是不断调用`next()`方法，每次遇到`yield x;`，返回一个对象{value: x, done: true/false}；二是直接用`for ... of`循环迭代generator对象
   - generator作用：用一个对象来保存状态；把异步回调代码变成"同步代码"