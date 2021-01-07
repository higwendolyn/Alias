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

### 原型继承
### class继承

## 参考文章

[廖雪峰面向对象编程](https://www.liaoxuefeng.com/wiki/1022910821149312/1023022043494624)

[Vue源码解析准备篇](https://www.jianshu.com/p/c914ccd498e7)