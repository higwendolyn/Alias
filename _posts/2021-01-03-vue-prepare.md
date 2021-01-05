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

这样是不是顿时感觉严谨多了？同时，代码意义更明确了。为啥这么说呢？ 之前看打包后的vue源码，其中看到观察者模式实现时由于没有类型十分难看懂，但是看了这个Flow版本的源码，感觉容易懂。 当然，如果想学习Flow更多的细节， 可以看看下面这个学习文档：

[Flow学习资料](https://zhuanlan.zhihu.com/p/26204569)