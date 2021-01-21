---
layout: post
title: "JS基础---继承"
---

对JS的继承有更深一步的理解，可以轻松掌握和JS继承相关的问题。

问题1：
<font style="color: #ec7907;">JS的继承到底有多少种实现方式？</font>

问题2：
<font style="color: #ec7907;">ES6的extends关键字是用哪种继承方式实现的？</font>

![image.png](../../../images/inherit1.png)

## 原型链继承

原型链继承是比较常见的继承方式之一，其中涉及构造函数、原型和实例

* 每一个构造函数都有一个原型对象
* 原型对象又包含一个指向构造函数的指针
* 而实例则包含一个原型对象的指针

```javascript
function Parent1() {
    this.name = 'parent1';
    this.play = [1, 2, 3]
}
function Child1() {
    this.type = 'child2';
}
Child1.prototype = new Parent1();

console.log(new Child1());

var s1 = new Child1();
var s2 = new Child1();
s1.play.push(4);
console.log(s1.play, s2.play); // 引用类型，内存空间共享的，当一个发生变化，另外一个也随之进行变化
```