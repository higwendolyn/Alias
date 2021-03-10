---
layout: post
title: "JS基础---继承"
category: 'JavaScript系列文章'
---

对JS的继承有更深一步的理解，可以轻松掌握和JS继承相关的问题。

问题1：
<font style="color: #ec7907;">JS的继承到底有多少种实现方式？</font>

问题2：
<font style="color: #ec7907;">ES6的extends关键字是用哪种继承方式实现的？</font>

![image.png](../../../images/inherit1.png)

## 原型与原型链

### 原型链

**创建对象**

用```obj.xxx```访问一个对象的属性时，JavaScript引擎先在当前对象上查找该属性，如果没有找到，就到其原型对象上找，如果还没有找到，就一直上溯到```Object.prototype```对象，最后，如果还没有找到，就只能返回```undefined```。

创建一个Array对象：

```javascript
var arr = [1, 2, 3];
```

其原型链是：

```
arr ----> Array.prototype ----> Object.prototype ----> null
```

``Array.prototype``定义了```indexOf()```、```shift()```等方法，因此你可以在所有的Array对象上直接调用这些方法。

创建一个函数：

```javascript
function foo() {
    return 0;
}
```

函数也是一个对象，它的原型链是：

```
foo ----> Function.prototype ----> Object.prototype ----> null
```

由于```Function.prototype```定义了```apply()```等方法，因此，所有函数都可以调用```apply()```方法。

如果原型链很长，那么访问一个对象的属性就会因为花更多的时间查找而变得更慢，因此要注意不要把原型链搞得太长。

---

**构造函数**

除了直接用```{ ... }```创建一个对象外，JavaScript还可以用一种构造函数的方法来创建对象。

```javascript
function Student(name) {
    this.name = name;
    this.hello = function () {
        alert('Hello, ' + this.name + '!');
    }
}
var xiaoming = new Student('小明');
xiaoming.name; // '小明'
xiaoming.hello(); // Hello, 小明!
```

<font style="color: red;">注意</font>

如果不写```new```，这就是一个普通函数，它返回```undefined```。但是，如果写了```new```，它就变成了一个构造函数，它绑定的```this```指向新创建的对象，并默认返回```this```，也就是说，不需要在最后写```return this;```。

新创建的```xiaoming```的原型链是：

```
xiaoming ----> Student.prototype ----> Object.prototype ----> null
```

所以，```xiaoming```的原型指向函数Student的原型。如果你又创建了```xiaohong```、```xiaojun```，那么这些对象的原型与```xiaoming```是一样的：

```
xiaoming ↘
xiaohong -→ Student.prototype ----> Object.prototype ----> null
xiaojun  ↗
```

用```new Student()```创建的对象还从原型上获得了一个```constructor```属性，它指向函数```Student```本身：

```javascript
xiaoming.constructor === Student.prototype.constructor; // true
Student.prototype.constructor === Student; // true

Object.getPrototypeOf(xiaoming) === Student.prototype; // true

xiaoming instanceof Student; // true
```

![image.png](../../../images/prepare2.png)

红色箭头是原型链。注意，```Student.prototype```指向的对象就是```xiaoming```、```xiaohong```的原型对象，这个原型对象自己还有个属性```constructor```，指向```Student```函数本身。

另外，函数```Student```恰好有个属性```prototype```指向```xiaoming```、```xiaohong```的原型对象，但是```xiaoming```、```xiaohong```这些对象可没有```prototype```这个属性，不过可以用```__proto__```这个非标准用法来查看。

现在我们就认为```xiaoming```、```xiaohong```这些对象“继承”自```Student```。

```javascript
xiaoming.name; // '小明'
xiaohong.name; // '小红'
xiaoming.hello; // function: Student.hello()
xiaohong.hello; // function: Student.hello()
xiaoming.hello === xiaohong.hello; // false
```

* ```xiaoming```和```xiaohong```各自的```name```不同，这是对的，否则我们无法区分谁是谁了。

* ```xiaoming```和```xiaohong```各自的```hello```是一个函数，但它们是两个不同的函数，虽然函数名称和代码都是相同的！

如果我们通过```new Student()```创建了很多对象，这些对象的```hello```函数实际上只需要共享同一个函数就可以了，这样可以节省很多内存。

要让创建的对象共享一个```hello```函数，根据对象的属性查找原则，我们只要把```hello```函数移动到```xiaoming```、```xiaohong```这些对象共同的原型上就可以了，也就是```Student.prototype```：

![image.png](../../../images/prepare3.png)

```javascript
function Student(name) {
    this.name = name;
}

Student.prototype.hello = function () {
    alert('Hello, ' + this.name + '!');
};
```
用new创建基于原型的JavaScript的对象就是这么简单！

### 原型继承

JavaScript由于采用原型继承，我们无法直接扩展一个Class，因为根本不存在Class这种类型。

上文，```Student```构造函数：

```javascript
function Student(props) {
    this.name = props.name || 'Unnamed';
}

Student.prototype.hello = function () {
    alert('Hello, ' + this.name + '!');
}
```

所以，```Student```的原型链：

![image.png](../../../images/prepare4.png)

基于```Student```扩展出```PrimaryStudent```，可以先定义出```PrimaryStudent```：

```javascript
function PrimaryStudent(props) {
    // 调用Student构造函数，绑定this变量:
    Student.call(this, props);
    this.grade = props.grade || 1;
}
```

调用了```Student```构造函数不等于继承了```Student```，```PrimaryStudent```创建的对象的原型是：

```
new PrimaryStudent() ----> PrimaryStudent.prototype ----> Object.prototype ----> null
```

必须想办法把原型链修改为：

```
new PrimaryStudent() ----> PrimaryStudent.prototype ----> Student.prototype ----> Object.prototype ----> null
```

这样新的基于```PrimaryStudent```创建的对象不但能调用```PrimaryStudent.prototype```定义的方法，也可以调用```Student.prototype```定义的方法。

如果你想用最简单粗暴的方法这么干：

```
PrimaryStudent.prototype = Student.prototype;
```

这样的话，```PrimaryStudent```和```Student```共享一个原型对象，那还要定义```PrimaryStudent```干啥？

我们必须借助一个中间对象来实现正确的原型链，这个中间对象的原型要指向```Student.prototype```。

```javascript
// PrimaryStudent构造函数:
function PrimaryStudent(props) {
    Student.call(this, props); // // 调用Student构造函数，绑定this变量:
    this.grade = props.grade || 1;
}

// 空函数F:
function F() {
}

// 把F的原型指向Student.prototype:
F.prototype = Student.prototype;

// 把PrimaryStudent的原型指向一个新的F对象，F对象的原型正好指向Student.prototype:
PrimaryStudent.prototype = new F();

// 以上三句也可改成
// PrimaryStudent.prototype = Object.Create(Student.prototype);

// 把PrimaryStudent原型的构造函数修复为PrimaryStudent:
PrimaryStudent.prototype.constructor = PrimaryStudent;

// 继续在PrimaryStudent原型（就是new F()对象）上定义方法：
PrimaryStudent.prototype.getGrade = function () {
    return this.grade;
};

// 创建xiaoming:
var xiaoming = new PrimaryStudent({
    name: '小明',
    grade: 2
});
xiaoming.name; // '小明'
xiaoming.grade; // 2

// 验证原型:
xiaoming.__proto__ === PrimaryStudent.prototype; // true
xiaoming.__proto__.__proto__ === Student.prototype; // true

// 验证继承关系:
xiaoming instanceof PrimaryStudent; // true
xiaoming instanceof Student; // true
```

新的原型链：

![image.png](../../../images/prepare5.png)

<font style="color: red;">注意</font>

函数```F```仅用于桥接，我们仅创建了一个```new F()```实例，而且，没有改变原有的```Student```定义的原型链。

如果把继承这个动作用一个```inherits()```函数封装起来，还可以隐藏F的定义，并简化代码：

```javascript
function inherits(Child, Parent) {
    var F = function () {};
    F.prototype = Parent.prototype;
    Child.prototype = new F();
    // 以上也可改成
    // Child.prototype = Object.Create(Parent.prototype);
    Child.prototype.constructor = Child;
}
```

这个```inherits()```函数可以复用：

```javascript
function Student(props) {
    this.name = props.name || 'Unnamed';
}

Student.prototype.hello = function () {
    alert('Hello, ' + this.name + '!');
}

function PrimaryStudent(props) {
    Student.call(this, props);
    this.grade = props.grade || 1;
}

// 实现原型继承链:
inherits(PrimaryStudent, Student);

// 绑定其他方法到PrimaryStudent原型:
PrimaryStudent.prototype.getGrade = function () {
    return this.grade;
};
```

---

**构造函数的继承**

* 构造函数绑定
* prototype模式
* 直接继承prototype
* 利用空对象作为中介
* 拷贝继承

<font style="color: #ec7907;">构造函数绑定</font>

最简单的方法，使用```call```或```apply```方法，将父对象的构造函数绑定在子对象上

```javascript
function Cat(name,color){
  Animal.apply(this, arguments);
  this.name = name;
  this.color = color;
}
var cat1 = new Cat("大毛","黄色");
alert(cat1.species); // 动物
```
---

<font style="color: #ec7907;">prototype模式</font>

如下：

```javascript
Cat.prototype = new Animal();  // 将Cat的prototype对象指向一个Animal的实例,
                               // 它相当于完全删除了prototype 对象原先的值，然后赋予一个新值
Cat.prototype.constructor = Cat;
var cat1 = new Cat("大毛","黄色");
alert(cat1.species); // 动物
```

任何一个```prototype```对象都有一个constructor属性，指向它的构造函数。

如果没有"Cat.prototype = new Animal();"这一行，```Cat.prototype.constructor```是指向```Cat```的；加了这一行以后，```Cat.prototype.constructor```指向```Animal```

```javascript
alert(Cat.prototype.constructor == Animal); //true
```

每一个实例也有一个constructor属性，默认调用```prototype```对象的```constructor```属性

```javascript
alert(cat1.constructor == Cat.prototype.constructor); // true
```

因此，在运行"Cat.prototype = new Animal();"这一行之后，```cat1.constructor```也指向```Animal```！

这显然会导致继承链的紊乱（cat1明明是用构造函数Cat生成的），因此我们必须手动纠正，将Cat.prototype对象的constructor值改为Cat。

---

<font style="color: #ec7907;">直接继承prototype</font>

第三种方法是对第二种方法的改进。由于Animal对象中，不变的属性都可以直接写入```Animal.prototype```。所以，我们也可以让Cat()跳过 Animal()，直接继承Animal.prototype。

```javascript
Cat.prototype = Animal.prototype;
Cat.prototype.constructor = Cat;  // 实际上把Animal.prototype对象的constructor属性也改掉了
var cat1 = new Cat("大毛","黄色");
alert(cat1.species); // 动物    
```

这样做的优点是效率比较高（不用执行和建立Animal的实例了），比较省内存。缺点是 ```Cat.prototype```和```Animal.prototype```现在指向了同一个对象，那么任何对Cat.prototype的修改，都会反映到Animal.prototype。

---

<font style="color: #ec7907;">利用空对象作为中介</font>

如下：

```javascript
var F = function(){}; // F是空对象，所以几乎不占内存
F.prototype = Animal.prototype;
Cat.prototype = new F();
Cat.prototype.constructor = Cat; // 修改Cat的prototype对象，就不会影响到Animal的prototype对象
```

封装函数

```javascript
function extend(Child, Parent) {
  var F = function(){};
  F.prototype = Parent.prototype;
  Child.prototype = new F();
  Child.prototype.constructor = Child;
  Child.uber = Parent.prototype; // 为子对象设一个uber属性，这个属性直接指向父对象的prototype属性; 等于在子对象上打开一条通道，可以直接调用父对象的方法
}
```

---

<font style="color: #ec7907;">拷贝继承</font>

简单说，如果把父对象的所有属性和方法，拷贝进子对象，不也能够实现继承

```javascript
function extend2(Child, Parent) {
  var p = Parent.prototype;
  var c = Child.prototype;
  for (var i in p) {
    c[i] = p[i]; // 用的是浅拷贝的方法,c[i] 是指向到 p[i], 而非赋值,建议使用深拷贝
  }
  c.uber = p;
}
```

### class继承

新的关键字```class```从ES6开始正式被引入到JavaScript中。```class```的目的就是让定义类更简单

用函数实现```Student```的方法：

```javascript
function Student(name) {
    this.name = name;
}

Student.prototype.hello = function () {
    alert('Hello, ' + this.name + '!');
}
```

用新的```class```关键字来编写```Student```，可以这样写：

```javascript
class Student {
    constructor(name) {
        this.name = name;
    }

    hello() {
        alert('Hello, ' + this.name + '!');
    }
}
```

比较一下就可以发现，```class```的定义包含了构造函数```constructor```和定义在原型对象上的函数```hello()```（注意没有function关键字），这样就避免了```Student.prototype.hello = function () {...}```这样分散的代码。

创建一个```Student```对象代码和前面章节完全一样：

```JS
var xiaoming = new Student('小明');
xiaoming.hello();
```

---

**class继承**

直接通过```extends```来实现:

```javascript
class PrimaryStudent extends Student {
    constructor(name, grade) {
        super(name); // 记得用super调用父类的构造方法!
        this.grade = grade;
    }

    myGrade() {
        alert('I am at grade ' + this.grade);
    }
}
```

<font style="color: red;">注意</font>

所以，```PrimaryStudent```的定义也是```class```关键字实现的，而```extends```则表示原型链对象来自```Student```。子类的构造函数可能会与父类不太相同，例如，PrimaryStudent需要name和grade两个参数，并且需要通过```super(name)```来调用父类的构造函数，否则父类的```name```属性无法正常初始化。

此外，```PrimaryStudent```已经自动获得了父类```Student```的hello方法，我们又在子类中定义了新的myGrade方法。

## 继承的几种实现
### 原型链继承

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

### 构造函数继承（借助call）

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

### 组合继承（前两种组合）

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

### 原型式继承

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

### 寄生式继承

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

### 寄生组合式继承

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
### ES6的 extends 关键字实现逻辑

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