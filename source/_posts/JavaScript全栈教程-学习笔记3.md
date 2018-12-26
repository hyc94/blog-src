---
title: JavaScript全栈教程-学习笔记3
date: 2018-12-26 20:35:51
tags: JavaScript
categories: 学习笔记
---

JavaScript全栈教程-学习笔记系列第三篇--浏览器对象

* 浏览器对象
* 操作DOM
* 操作表单
* 操作文件
* AJAX
* Promise
* Canvas

------

<!-- more-->

## 浏览器对象

1. `window`对象不但充当全局作用域，而且表示浏览器窗口。`window`对象有`innerWidth`和`innerHeight`属性，可以获取窗口的内部宽度和高度，除去菜单栏、工具栏、边框的网页净宽高；还有`outerWidth`和`outerHeight`属性，可以获取浏览器窗口的整个宽高。

2. `navigator`对象表示浏览器的信息：

   * navigator.appName：浏览器名称
   * navigator.appVersion：浏览器版本
   * navigator.language：浏览器设置的语言
   * navigator.platform：操作系统类型
   * navigator.userAgent：浏览器设定的User-Agent字符串

   针对不同浏览器编写不同的代码，不要用`if`判断浏览器版本，可以使用短路运算符计算`||`

3. `screen`对象表示屏幕的信息：
   * screen.width：屏幕宽度，以像素为单位
   * screen.height：屏幕高度，以像素为单位
   * screen.colorDepth：返回颜色位数，如8、16、24

4. `location`对象表示当前页面的URL信息：

   ```javascript
   location.href; //http://www.example.com:8080/path/index.html?a=1&b=2#TOP
   location.protocol; //http
   location.host; //www.example.com
   location.port; //端口
   location.pathname; //path/index.html
   location.search; //?a=1&b=2
   location.hash; //TOP
   location.assign('/'); //加载一个新页面
   location.reload(); //重新加载当前页面
   ```

5. `document`对象表示当前页面，`document`对象就是整个DOM树的根节点

   `document`的`title`属性是从HTML文档中的`<title>xxx</title>`读取，但是可以动态改变

   `document`对象提供的`getElementById()`和`getElementsByTagName()`可以按ID获得一个DOM节点和按Tag名称获得一组DOM节点

   `document.cookie`读取到当前页面的Cookie，所以存在引入的第三方JavaScript存在恶意代码的风险，服务器设置Cookie时可以使用`httpOnly`，设定了`httpOnly`的Cookie将不能被JavaScript读取

6. `history`对象保存了浏览器的历史记录，JavaScript可以调用`history`对象的`back()`或`forward()`，相当于用户点击了浏览器的“后退”或“前进”按钮，对于现代Web页面，不应该使用`history`这个对象。

## 操作DOM

