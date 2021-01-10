---
layout: post
title: "Vue源码解析准备篇"
---

之前有尝试看过Vue源码，但是理解都不是很深刻动不动就忘记了，现在来把前置条件学习一下。

![image.png](../../../images/prepare1.png)

## Flow基本语法

javascript是弱类型的语言，在写代码灰常爽的同时也十分容易犯错误，所以Facebook搞了这么一个类型检查工具，可以加入类型的限制，提高代码质量，举个例子:
```
function sum(a, b) {
  return a + b;
}
```

如果这么调用这个函数sum('a', 1) 甚至sum(1, [1,2,3])这么调用，执行时会得到一些你想不到的结果，这样编程未免太不稳定了。那我们看看用了Flow之后的结果：
```
function sum(a: number, b:number) {
  return a + b;
}
```

多了一个number的限制，标明对a和b只能传递数字类型的，否则的话用Flow工具检测会报错。

其实这里大家可能有疑问，这么写还是js吗？ 浏览器还能认识执行吗？当然不认识了，所以需要翻译或者说编译。其实现在前端技术发展太快了，各种插件层出不穷--Babel、Typescript等等，其实都是将一种更好的写法编译成浏览器认识的javascript代码（我们以前都是写浏览器认识的javascript代码的）。

我们继续说Flow的事情，在Vue源码中其实出现的Flow语法都比较好懂，比如下面这个函数的定义:
```
export function renderList (
  val: any,
  render: (
    val: any,
    keyOrIndex: string | number,
    index?: number
  ) => VNode
): ?Array<VNode>{
...
}
```

val是any代表可以传入的类型是任何类型;

keyOrIndex是```string|number```类型，代表要不是string类型，要不是number，不能是别的;

index?:number这个我们想想正则表达式中？的含义---0个或者1个，这里其实意义也是一致的，但是要注意?的位置是在冒号之前还是冒号之后--因为这两种可能性都有，上面代码中**问号是跟在冒号前面，代表index可以不传，但是传的话一定要传入数字类型**；如果**问号是在冒号后面的话，则代表这个参数必须要传递，但是可以是数字类型也可以是空**。

如果想学习Flow更多的细节， 可以看看下面这个学习文档：

[Flow学习资料](https://zhuanlan.zhihu.com/p/26204569)

## 原型与原型继承

### 原型链

**创建对象**

用```obj.xxx```访问一个对象的属性时，JavaScript引擎先在当前对象上查找该属性，如果没有找到，就到其原型对象上找，如果还没有找到，就一直上溯到```Object.prototype```对象，最后，如果还没有找到，就只能返回```undefined```。

创建一个Array对象：

```
var arr = [1, 2, 3];
```

其原型链是：

```
arr ----> Array.prototype ----> Object.prototype ----> null
```

``Array.prototype``定义了```indexOf()```、```shift()```等方法，因此你可以在所有的Array对象上直接调用这些方法。

创建一个函数：
```
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

```JavaScript
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

```JavaScript
xiaoming.constructor === Student.prototype.constructor; // true
Student.prototype.constructor === Student; // true

Object.getPrototypeOf(xiaoming) === Student.prototype; // true

xiaoming instanceof Student; // true
```

![image.png](../../../images/prepare2.png)

红色箭头是原型链。注意，```Student.prototype```指向的对象就是```xiaoming```、```xiaohong```的原型对象，这个原型对象自己还有个属性```constructor```，指向```Student```函数本身。

另外，函数```Student```恰好有个属性```prototype```指向```xiaoming```、```xiaohong```的原型对象，但是```xiaoming```、```xiaohong```这些对象可没有```prototype```这个属性，不过可以用```__proto__```这个非标准用法来查看。

现在我们就认为```xiaoming```、```xiaohong```这些对象“继承”自```Student```。

```JavaScript
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

```JavaScript
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

```
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

```
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

```JavaScript
// PrimaryStudent构造函数:
function PrimaryStudent(props) {
    Student.call(this, props);
    this.grade = props.grade || 1;
}

// 空函数F:
function F() {
}

// 把F的原型指向Student.prototype:
F.prototype = Student.prototype;

// 把PrimaryStudent的原型指向一个新的F对象，F对象的原型正好指向Student.prototype:
PrimaryStudent.prototype = new F();

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

```JavaScript
function inherits(Child, Parent) {
    var F = function () {};
    F.prototype = Parent.prototype;
    Child.prototype = new F();
    Child.prototype.constructor = Child;
}
```

这个```inherits()```函数可以复用：

```JavaScript
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

```JavaScript
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

```JavaScript
　　Cat.prototype = new Animal();  // 将Cat的prototype对象指向一个Animal的实例,
                                  // 它相当于完全删除了prototype 对象原先的值，然后赋予一个新值

　　Cat.prototype.constructor = Cat;

　　var cat1 = new Cat("大毛","黄色");

　　alert(cat1.species); // 动物
```

任何一个```prototype```对象都有一个constructor属性，指向它的构造函数。

如果没有"Cat.prototype = new Animal();"这一行，```Cat.prototype.constructor```是指向```Cat```的；加了这一行以后，```Cat.prototype.constructor```指向```Animal```

```JavaScript
alert(Cat.prototype.constructor == Animal); //true
```

每一个实例也有一个constructor属性，默认调用```prototype```对象的```constructor```属性

```JavaScript
alert(cat1.constructor == Cat.prototype.constructor); // true
```

因此，在运行"Cat.prototype = new Animal();"这一行之后，```cat1.constructor```也指向```Animal```！

这显然会导致继承链的紊乱（cat1明明是用构造函数Cat生成的），因此我们必须手动纠正，将Cat.prototype对象的constructor值改为Cat。

---

<font style="color: #ec7907;">直接继承prototype</font>

第三种方法是对第二种方法的改进。由于Animal对象中，不变的属性都可以直接写入```Animal.prototype```。所以，我们也可以让Cat()跳过 Animal()，直接继承Animal.prototype。

```JavaScript
   Cat.prototype = Animal.prototype;

　　Cat.prototype.constructor = Cat;  // 实际上把Animal.prototype对象的constructor属性也改掉了

　　var cat1 = new Cat("大毛","黄色");

　　alert(cat1.species); // 动物    
```

这样做的优点是效率比较高（不用执行和建立Animal的实例了），比较省内存。缺点是 ```Cat.prototype```和```Animal.prototype```现在指向了同一个对象，那么任何对Cat.prototype的修改，都会反映到Animal.prototype。

<font style="color: #ec7907;">利用空对象作为中介</font>

如下：

```JavaScript
　　var F = function(){}; // F是空对象，所以几乎不占内存

　　F.prototype = Animal.prototype;

　　Cat.prototype = new F();

　　Cat.prototype.constructor = Cat; // 修改Cat的prototype对象，就不会影响到Animal的prototype对象
```

封装函数

```JavaScript
　　   function extend(Child, Parent) {

　　　　var F = function(){};

　　　　F.prototype = Parent.prototype;

　　　　Child.prototype = new F();

　　　　Child.prototype.constructor = Child;

　　　　Child.uber = Parent.prototype; // 为子对象设一个uber属性，这个属性直接指向父对象的prototype属性; 等于在子对象上打开一条通道，可以直接调用父对象的方法

　　}
```


<font style="color: #ec7907;">拷贝继承</font>

简单说，如果把父对象的所有属性和方法，拷贝进子对象，不也能够实现继承

```JavaScript
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



## 参考文章

[廖雪峰面向对象编程](https://www.liaoxuefeng.com/wiki/1022910821149312/1023022043494624)

[Vue源码解析准备篇](https://www.jianshu.com/p/c914ccd498e7)