---
layout: post
title: "JS基础---闭包难点剖析"
category: 'JavaScipt'
---

JS中的闭包是相当重要的概念，本节重点讲解闭包。

## 闭包

JavaScript 中的闭包是相当重要的概念，并且与作用域相关知识的指向密切相关。

问题1：
<font style="color: #ec7907;">JavaScript 中的作用域是什么意思？</font>

问题2：
<font style="color: #ec7907;">闭包会在哪些场景中使用？</font>

问题3：
<font style="color: #ec7907;">通过定时器循环输出自增的数字通过 JS 的代码如何实现？</font>

## 作用域基本介绍

JavaScript 的作用域：指变量能够被访问到的范围

<mark style="background: #00999970;">ES5 之前</mark> 全局作用域和函数作用域

<mark style="background: #e1944747;">ES6 出现之后</mark> 新增块级作用域

### 全局作用域

变量一般分为全局变量和局部变量两种。

**全局变量**：是挂载在 window 对象下的变量，所以在网页的任何位置你都可以使用并且访问到这个全局变量

```javascript
var globalName = 'global';

function getName() {
    console.log(globalName); // global
    var name = 'inner';
    console.log(name); // inner
}

getName();
console.log(name); //
console.log(globalName); // global

function setName() {
    vName = 'setName'; // 没有定义就被赋值，默认是全局变量
}

setName();
console.log(vName); // setName
console.log(window.vName); // setName
```

定义很多全局变量的时候，会容易引起变量命名的冲突。

### 函数作用域

```javascript
function getName() {
    var name = 'inner';
    console.log(name); // inner
}

getName();
console.log(name);
```

除了这个函数内部，其他地方都是不能访问到它的，当这个函数被执行完之后，这个局部变量也相应会被销毁。

### 块级作用域

最直接的表现就是新增的 let 关键词

使用 let 关键词定义的变量只能在块级作用域中被访问

<mark>暂时性死区</mark>这个变量在定义之前是不能被使用的

if 语句以及 for 语句后面 {...} 这里面所包括的就是块级作用域

```javascript
console.log(a); // a is not defined
if (true) {
    let a = '123';
    console.log(a); // 123
}
console.log(a); // a is not defined
```

## 什么是闭包？

<mark style="background: #e1944796;">红宝书闭包的定义：闭包是指有权访问另外一个函数作用域中的变量的函数。</mark>

<mark style="background: #e1c94799;">MDN：一个函数和对其周围状态的引用捆绑在一起（或者说函数被引用包围），这样的组合就是闭包（closure）
也就是说，闭包让你可以在一个内层函数中访问到其外层函数的作用域</mark>

### 闭包的基本概念

闭包其实就是一个可以访问其他函数内部变量的**函数**

通常情况下，函数内部变量是无法在外部访问的，因此使用闭包的左右就是具备实现了能在外部访问某个函数内部变量的功能。

```javascript
function fun1() {
    var a = 1;
    return function() {
        console.log(a);
    }
}

fun1();
var result = fun1();
result(); // 1
```

### 闭包产生的原因

作用域链的基本概念：

当访问一个变量时，代码解释器会首先在当前的作用域查找

如果没有找到，就去父级作用域去查找

直到找到该变量或者不存在父级作用域中

```javascript
var a = 1;
function fun1() {
    var a = 2;
    function fun2() {
        var a = 3;
        console.log(a); // 3
    }
}
```

闭包的本质：当前环境中存在指向父级作用域的引用。

```javascript
function fun1() {
    var a = 2;
    function fun2() {
        console.log(a); // 2
    }
    return fun2;
}
var result = fun1();
result;
```

是不是只有返回函数才算是产生了闭包呢？

不是，只需要让父级作用域的引用存在即可

```javascript
var fun3;
function fun1() {
    var a = 2;
    fun3 = function() {
        console.log(a);
    }
}
fun1();
fun3();
```

### 闭包的表现形式

1. 返回一个函数，见上文闭包产生原因内容
2. 在定时器、事件监听、Ajax请求、Web Workers或者任何异步中只要使用了回调函数，实际上就是在使用闭包

```javascript
// 定时器
setTimeout(function handler() {
    console.log('1');
}, 1000);
// 事件监听
```