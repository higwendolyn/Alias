---
layout: post
title: "JS基础---JS异步编程的核心 Promise"
category: 'JavaScript'
---

ES6 之前，社区就提出了 Promise 的方案，后随着ES6 将其加入进去，统一了方法，提供了原生的 Promise 对象。

问题1：
<font style="color: #ec7907;">Promise 内部究竟有几种状态？</font>

问题2：
<font style="color: #ec7907;">Promise 是怎么解决回调地狱问题的？</font>

## Promise 的基本情况

简单来说，它就是一个容器，里面保存着某个未来才会结束的事件的结果

从语法上说，Promise是一个对象，从它可以获取异步操作的消息

```javascript
function read(url) {
    return new Promise((resolve, reject) => {
        fs.readFile(url, 'utf8', (err, data) => {
            if (err) reject(err);
            resolve(data);
        });
    });
}
read(A).then(data => {
    return read(B);
}).then(data => {
    return read(C);
}).then(data => {g
    return read(D);
}).catch(reason => {
    console.log(reason);
});
```

* 待定(pending): 初始状态，既没有被完成，也没有被拒绝
* 已完成(fullfilled): 操作成功完成
* 已拒绝(rejected): 操作失败

待定状态的 Promise 对象执行的话，最后要么会通过一个值完成，要么会通过一个原因被拒绝

> 最后Promise.prototype.then 和Promise.prototype.catch 方法返回的是一个 Promise 所以它们可以继续被链式调用

图片1

深入理解，可学习"有限状态机"

## Promise 如何解决回调地狱

回调地狱问题：
* 多层嵌套的问题
* 每种任务的处理结果存在两种可能性（成功或失败），那么需要在每种任务执行结束后分别处理这两种可能性

Promise 利用了三大技术手段来解决回调地狱：
回调函数延迟绑定、返回值穿透、错误冒泡


回调函数延迟绑定

```javascript
let readFilePromise = (filename) => {
    fs.readFile(filename, (err, data) => {
        if (err) {
            reject(err);
        } else {
            resolve(data);
        }
    })
}
readFilePromise('1.json').then(data => {
    return readFilePromise('2.json');
})
```

返回值穿透

```javascript
let x = readFilePromise('1.json').then(data => {
    return readFilePromise('2.json') // 这是返回的Promise
});
x.then(/* 内部逻辑省略 */)
```

错误冒泡

```javascript
readFilePromise('1.json').then(data => {
    return readFilePromise('2.json');
}).then(data => {
    return readFilePromise('3.json');
}).then(data => {g
    return readFilePromise('4.json');
}).catch(err => {
    // ...
});
```

## Promise 的静态方法

从语法、参数以及方法的代码几个方面来分别介绍 all、allSettled、any、race 这四种方法

### all 方法

* 语法：Promise.all(iterable)
* 参数：一个可迭代对象，如Array
* 描述：此方法对于汇总多个Promise的结果很有用，在ES6中可以将多个Promise.all异步请求并行操作

1. 当所有结果成功返回时按照请求顺序返回成功
2. 当其中有一个失败方法时，则进入失败方法

```javascript
// 获取
```