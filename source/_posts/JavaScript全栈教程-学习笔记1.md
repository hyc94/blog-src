---
title: JavaScript全栈教程-学习笔记1
date: 2018-11-20 21:51:24
tags: JavaScript
categories: 前端
---

JavaScript全栈教程-学习笔记系列第一篇

* 快速入门
* 函数
* 标准对象

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

## 标准对象

1. `null`、`Array`的类型也是`object`，无法用`typeof`区分出`null`、`Array`和普通的object--`{}`

2. 包装对象用`new`创建，包装对象和原始值用`===`比较会返回`false`，所以尽量不要使用包装对象，尤其是针对`string`类型

3. 如果使用`Number`、`Boolean`和`String`时没有写`new`，`Number()`、`String()`等会被当做普通函数，把数据转换为`number`、`boolean`和`string`类型

4. 用`parseInt()`或`parseFloat()`来转换任意类型到`number`；用`String()`来转换任意类型到`string`，或者调用对象的`toString()`方法，注意`number`对象调用`toString()`报SyntaxError，要特殊处理一下：

   ```javascript
   123..toString();  //'123'，注意是两个点！
   
   (123).toString();
   ```

5. `typeof`操作符可以判断出`number`、`boolean`、`string`、`function`、`undefined`；判断`Array`要使用`Array.isArray(arr)`；判断`null`要使用`myVar === null`

6. 判断全局变量是否存在`typeof window.myVar === 'undefined'`；函数内部变量是否存在使用`typeof myVar === 'undefined'`

7. Date对象月份值从0开始，0=1月...，11=12月

8. 创建Date对象的3种方式：

   ```javascript
   var d1 = new Date(2015, 5, 19, 20, 15, 30, 123)
   
   var d2 = Date.parse('2015-06-24T19:49:22.875+08:00')
   
   var d3 = new Date(1435146562875)
   ```

   使用Date.parse()时传入的字符串使用实际月份01~12，转换成Date对象使用getMonth()获取的月份是0~11

9. `d.toLocaleString()`：本地时间，显示的字符串与操作系统设定的格式有关，`d.toUTCString()`：UTC时间

10. 用`\d`可以匹配一个数字，`\w`可以匹配一个字母或数字，`.`可以匹配任意字符，`\s`可以匹配一个空格（包括Tab等空白符）；用`*`表示任意个字符（包括0个），用`+`表示至少一个字符，用`?`表示0个或1个字符，用`{n}`表示n个字符，用`{n, m}`表示 n-m个字符

11. 可以用`[]`表示范围，`A|B`可以匹配A或B，`^`表示行的开都，`$`表示行的开头

12. 有两种方式创建一个正则表达式：第一种是直接通过`/正则表达式/`，第二种是通过`new RegExp('正则表达式')`创建一个RegExp对象，RegExp对象的`test()`方法用于测试给定字符串是否符号条件

13. 用正则表达式可以切分字符串；正则表达式还有提取子串的功能，用`()`表示要提取的分组

    ```javascript
    var re = /^(\d{3})-(\d{3,8})$/;
    re.exec('010-12345'); // ['010-12345', '010', '12345']
    re.exec('010 12345'); // null
    ```

    组可以用`RegExp`对象的`exec()`方法提取出子串：匹配成功后，会返回一个`Array`，第一个元素是匹配到的整个字符串，后面的字符串表示匹配成功的子串；匹配失败会返回`null`

14. 正则匹配默认是贪婪匹配，加个`?`可以采用非贪婪匹配

    ```javascript
    var re = /^(\d+)(0*)$/;
    re.exec('102300'); // ['102300', '102300', '']
    
    var re = /^(\d+?)(0*)$/;
    re.exec('102300'); // ['102300', '1023', '00']
    ```

15. 还有几个特殊标志，`g`表示全局搜索，可以执行`exec()`多次搜索；指定`i`标志，表示忽略大小写，`m`标志，表示执行多行匹配
16. JSON序列化：`JSON.stringify(obj, null, '   ')`，第二个参数用于筛选对象的键值，可以传入`Array`，还可以传入函数，每个键值都会被函数处理，第三个参数调整格式，按缩进输出。还可以给obj对象定义一个`toJSON()`的方法
17. JSON反序列化：`JSON.parse()`把字符串转化成对象，还可以接收一个函数，用来转换解析出的参数