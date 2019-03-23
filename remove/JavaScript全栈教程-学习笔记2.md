---
title: JavaScript全栈教程-学习笔记2
date: 2018-12-24 20:27:48
tags: JavaScript
categories: 学习笔记
---

JavaScript全栈教程-学习笔记系列第二篇

* 面向对象编程

------

<!-- more -->

## 面向对象编程

1. JavaScript不区分类和实例的概念，而是通过原型（prototype）来实现面向对象编程

   ```javascript
   subclass.__proto__ = superclass
   ```

2. 不要直接用`obj.__proto__`去改变一个对象的原型，并且低版本的IE也无法使用`__proto__`;`Object.create()`方法可以传入一个原型对象，并创建一个基于该原型的新对象，但是新对象什么属性都没有

3. 当使用`obj.xxx`访问对象属性，JavaScript引擎先在当前对象查找，如果没有找到，就到原型对象上找，如果还没有找到，就一直上溯到`Object.prototype`对象，最后如果没找到，就只能返回`undefined`

   数组原型链：

   ```javascript
   arr ——-> Array.propotype ---> Object.prototype ---> null
   ```

   函数原型链

   ```javascript
   foo ---> Function.prototype ---> Object.prototype ---> null
   ```

4. JavaScript还可以用构造函数来创建对象

   ```javascript
   function Student(name) {
       this.name = name;
       this.hello = function() {
           alert('Hello ' + this.name + '!');
       }
   }
   
   //如果不写new，就是一个普通函数，返回undefined；写了new，它绑定的this指向新创建的对象，并默认返回this
   var xiaoming = new Student('小明'); 
   xiaoming.name //‘小明’
   
   //xiaoming原型链：xiaoming ---> Student.prototype ---> Object.prototype ---> null
   //new Student()创建的对象还从原型上获得一个constructor属性，它指向函数Student本身
   xiaoming.constructor === Student.prototype.constructor; //true
   Student.prototype.constructor == Student; //true
   Object.getPrototypeOf(xiaoming) === Student.prototype; //true
   xiaoming instanceof Student; //true
   ```

   {% asset_img JavaScript教程2-1.png JavaScript教程2 %}

5. 我们可以编写一个`createObj()`函数，在内部封装所有`new`操作，一个常用的编程模式如下：

   ```javascript
   function Student(props) {
       this.name = props.name || '匿名'; //默认值为'匿名'
       this.grade = props.grade || 1; //默认值为1
   }
   
   Student.prototype.hello = function() {
       alert('Hello ' + this.name + '!');
   };
   
   function createStudent(props) {
       return new Student(props || {})
   }
   ```

   这个`createObj()`函数的有点：一是不需要`new`来调用，二是参数非常灵活

6. JavaScript的原型继承实现方式就是：

   1. 定义新的构造函数，并在内部用`call()`调用希望“继承”的构造函数，并绑定`this`；
   2. 借助中间函数`F`实现原型链继承，最好通过封装`inherits`函数完成；
   3. 继续在新的构造函数的原型上定义新方法；

   ```javascript
   function inherits(Child, Parent) {
       var F = function() {};
       F.prototype = Parent.prototype;
       Child.prototype = new F();
       Child.prototype.constructor = Child;
   }
   
   function Student(props) {
       this.name = props.name || 'Unnamed';
   }
   
   Student.prototype.hello = function() {
       alert('Hello ' + this.name + '!');
   }
   
   function PrimaryStudent(props) {
       Student.call(this, props);
       this.grade = props.grade || 1;
   }
   
   // 实现原型继承链
   inherits(PrimaryStudent, Student);
   
   // 绑定其他方法到PrimaryStudent原型：
   PrimaryStudent.prototype.getGrade = function () {
       return this.grade;
   }
   ```

7. 新的关键字`class`从ES6开始正式被引入到JavaScript中，`class`的定义包含了构造函数`constructor`和定义在原型对象上的函数`hello()`（注意没有`function`关键字）

   ```javascript
   class Student {
       constructor(name) {
           this.name = name;
       }
       
       hello() {
           alert('Hello ' + this.name + '!');
       }
   }
   ```

8. 用`class`定义对象的另一个巨大的好处就是继承更方便，直接通过`extends`来实现：

   ```javascript
   class PrimaryStudent extends Student {
       constructor(name, grade) {
           super(name); //记得用super调用父类的构造器
           this.grade = grade;
       }
       
       myGrade() {
           alert('I am at grade ' + this.grade);
       }
   }
   ```
