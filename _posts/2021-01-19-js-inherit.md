---
layout: post
title: "JS基础---继承"
category: 'JavaScript'
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
    this.type = 'child1';
}
Child1.prototype = new Parent1();

console.log(new Child1());

var s1 = new Child1();
var s2 = new Child1();
s1.play.push(4);
console.log(s1.play, s2.play); // 引用类型，内存空间共享的，当一个发生变化，另外一个也随之进行变化
```

## 构造函数继承（借助call）

```javascript
function Parent1() {
    this.name = 'parent1';
}
Parent1.prototype.getName = function (){
    return this.name;
}

function Child1() {
    Parent1.call(this);
    this.type = 'child1';
}

let child = new Child1();
console.log(child); // 没问题
console.log(child.getName()); // 会报错，只能继承父类的实例属性或者方法，不能继承原型属性或者方法
```

## 组合继承（前两种组合）

```javascript
function Parent3() {
    this.name = 'parent3';
    this.play = [1, 2, 3];
}

Parent3.prototype.getName = function (){
    return this.name;
}

function Child3() {
    // 第二次调用Paernt3()
    Parent3.call(this);
    this.type = 'child3';
}

// 第一次调用Parent3()
Child3.protptype = new Parent3();
// 手动挂上构造器，指向自己的构造函数
Child3.protptype.constructor = Child3;
var s3 = new Child3();
var s4 = new Child3();
s3.play.push(4);
console.log(s3.play, s4.play); // 不互相影响
console.log(s3.getName()); // 正常输出'parent3'
console.log(s4.getName()); // 正常输出'parent3'
```

## 原型式继承

```javascript
let parent4 = {
    name: 'parent4',
    friends: ['p1', 'p2', 'p3'],
    getName: function() {
        return this.name;
    }
};

let person4 = Object.create(parent4);
person4.name = 'tom';
person4.friends.push('jerry');

let person5 = Object.create(parent4);
person5.friends.push('lucy');

console.log(person4.name);
console.log(person4.name === person4.getName());
console.log(person5.name);
console.log(person4.friends);
console.log(person5.friends);
```

## 寄生式继承

使用原型式继承可以获得一份目标对象的浅拷贝，然后利用这个浅拷贝的能力再进行增强添加一些方法

寄生式继承相比于原型式继承，还是在父类基础上添加了更多的方法

```javascript
let parent5 = {
    name: 'parent4',
    friends: ['p1', 'p2', 'p3'],
    getName: function() {
        return this.name;
    }
};

function clone(original) {
    let clone = Object.create(original);
    clone.getFriends = function() {
        return this.friends;
    }
    return clone;
}

let person5 = clone(parent5);

console.log(person5.getName());
console.log(person5.getFriends());
```

## 寄生组合式继承

最优的继承方式

```javascript
function clone(parent, child)) {
    // 这里改用Object.create就可以减少组合继承中多进行一次构造的过程
    child.prototype = Object.create(parent.prototype);
    child.prototype.constructor = child;
}

function Parent6() {
    this.name = 'parent6';
    this.play = [1, 2, 3];
}
Parent6.prototype.getName = function() {
    return this.name;
}
function Child6() {
    Parent6.call(this);
    this.friends = 'child5';
}

clone(Parent6, Child6);

Child6.prototype.getFriends = function() {
    return this.friends;
}

let person6 = new Child6();
console.log(person6);
console.log(person6.getName());
console.log(person6.getFriends());
```
## ES6的 extends 关键字实现逻辑

使用关键词很容易直接实现JavaScript的继承，但是如果深入了解extends语法糖是怎么实现

```javascript
class Person {
    constructor(name) {
        this.name = name
    }
    // 原型方法
    // 即 Person.prototype.getName = function() {}
    // 下面可简写为getName() {...}
    getName = function() {
        console.log('Person:', this.name);
    }
}

class Gamer extends Person {
    constructor(name, age) {
        // 子类中存在构造函数，则需要在使用“this”之前首先调用super()。
        super(name)
        this.age = age
    }
}
const asuna = new Gamer('Asuna', 20)
asuna.getName() // 成功访问到父类的方法
```

编译后代码就是寄生组合式继承