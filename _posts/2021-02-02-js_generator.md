---
layout: post
title: "JS基础---理解Generator、Async/await 等异步编程的语法糖"
category: 'JavaScript系列文章'
---

深入理解Generator、Async/await 等异步编程的语法糖

问题1：
<font style="color: #ec7907;">Generator 执行之后，最后返回的是什么？</font>

问题2：
<font style="color: #ec7907;">async/await 的方式比 Promise 和 Generator 好在哪里？</font>

## Generator 基本介绍

Generator (生成器)是ES6 的新关键词

Generator 是一个带星号的“函数”，可以配合yield 关键词来暂停或者执行函数

```javascript
function* gen() {
    console.log('enter');
    let a = yield 1;
    let b = yield(function () {return 2})();
    return 3;
}
var g = gen() // 阻塞住，不会执行任何语句
console.log(typeof g) // 返回object 这里不是'function'
console.log(g.next())
console.log(g.next())
console.log(g.next())
console.log(g.next())

// output:
// {value:1,done:false}
// {value:2,done:false}
// {value:3,done:true}
// {value:undefined,done:true}
```

1. 调用gen(）函数后，程序会阻塞住，不会执行任何语句
2. 调用g.next()后，程序基础执行，直到遇到yield关键词时执行暂停
3. 一直执行next方法，最后返回一个对象，其存在两个属性：value和done

### yield基本介绍

yield同样是ES6的新关键词，配合Generator执行以及暂停

yield关键词最后返回一个迭代器对象，该对象有<mark>value</mark>和<mark>done</mark>两个属性，代表返回值以及是否完成

```javascript
function* gen1() {
    yield 1;
    yield* gen2();
    yield 4;
}

function* gen2() {
    yield 2;
    yield 3;
}

var g = gen1();
console.log(g.next())
console.log(g.next())
console.log(g.next())
console.log(g.next())

// output:
// {value:1,done:false}
// {value:2,done:false}
// {value:3,done:false}
// {value:4,done:false}
// {value:undefined,done:true}
```

<font style="color: #ec7907;">Generator 和异步编程有什么联系？怎么才可以把Generator 函数按照顺序一次性执行完呢</font>

### thunk 函数介绍

```javascript
let isString = (obj) => {
    return Object.prototype.toString.call(obj) == '[object String]';
};
let isFunction = (obj) => {
    return Object.prototype.toString.call(obj) == '[object Function]';
};
let isArray = (obj) => {
    return Object.prototype.toString.call(obj) == '[object Array]';
};

...
```

可抽象成一个函数

```javascript
let isType = (type) => {
    return (obj) => {
        return Object.prototype.toString.call(obj) == '[object ${type}]';
    }
};

let isString = isType('String');
let isArray = isType('Array');

isString('123'); // true
isArray([1, 2, 3]); // true
```

thunk函数的基本思路都是接收一定的参数，会生产出定制化的函数，最后使用定制化的函数去完成想要实现的功能

### Generator 和 thunk 结合

```javascript
const readFileThunk = (filename) => {
    return (callback) => {
        fs.readFile(filename, callback);
    }
}

const gen = function* () {
    const data1 = yield readFileThunk('1.txt')
    console.log(data1.toString())
    const data2 = yield readFileThunk('2.txt')
    console.log(data2.toString())
}

let g = gen();
g.next().value((err, data1) => {
    g.next(data1).value((err, data2) => {
        g.next(data2);
    })
})
// 也可以这么写
function run(gen){
    const next = (err, data) => {
        let res = gen.next(data);
        if (res.done) return;
        res.value(next);
    }
    next();
}
run(g);
```

```javascript
// 最后包装成Promise对象进行返回
const readFilePromise = (filename) => {
    return new Promise((resolve, reject) => {
        fs.readFile(filename, (err, data) => {
            if (err) {
                reject(err);
            } else {
                resolve(data);
            }
        })
    }).then(res => res);
}

// 这块和上面thunk的方式一样
const gen = function* () {
    const data1 = yield readFileThunk('1.txt')
    console.log(data1.toString())
    const data2 = yield readFileThunk('2.txt')
    console.log(data2.toString())
}

// 这块和上面thunk的方式一样
function run(gen){
    const next = (err, data) => {
        let res = gen.next(data);
        if (res.done) return;
        res.value(next);
    }
    next();
}
run(g);
```

### co函数库

**co函数库是一个小工具，用于处理Generator函数的自动执行**

> 核心原理是上面讲的通过和thunk函数以及Promise对象进行配合，包装成一个库

```javascript
const co = require('co');
let g = gen();
co(g).then(res => {
    console.log(res);
})
```

* Generator 函数就是一个异步操作的容器，co函数接受Generator函数作为参数并最后返回一个Promise对象
* 在返回的Promise对象里面，co先检查参数gen是否为Generator函数。如果是，就执行该函数；如果不是就返回，并将Promise对象的状态改为resolved
* co将Generator函数的内部指针对象的next方法，包装成onFulfilled函数。这主要是为了能够捕捉抛出的错误
* 关键的是next函数，它会反复调用自身

> 有时间记得看看co函数库的源码！！！

## async/await 介绍

> JS 的异步编程从最开始的回调函数的方式，演化到使用Promise对象，再到Generator + co函数的方式，每次都有一些改变，但又让人觉得不彻底

<mark>async/await 被称为 JS 中异步终极解决方案</mark>

既能够像co + Generator一样用同步的方式来书写异步代码，又得到底层的语法支持，无须借助任何第三方库

```javascript
// readFilePromise 依旧返回Promise对象
const readFilePromise = (filename) => {
    retuen new Promise((resolve, reject) => {
        fs.readFile(filename, (err, data) => {
            if (err) {
                reject(err);
            } else {
                resolve(data);
            }
        })
    }).then(res => res);
}

// 这里把Generator的*换成async，把yield换成await
const gen = async function() {
    const data1 = await readFilePromise('1.txt');
    console.log(data1.toString());
    const data2 = await readFilePromise('2.txt');
    console.log(data2.toString());
}
```

<span style="color: #e0650b;">内置执行器：</span>Generator 函数的执行必须靠执行器，因为不能一次性执行完成，所以之后才有了开源的co函数库

<span style="color: #234abf;">适用性更好：</span>co 函数库有条件约束，但是async函数的await关键词后面，可以不受约束

<span style="color: #28bb64;">可读性更好：</span>async 和 await，比起使用 * 号和 yield，语义更清晰明了

```javascript
async function func() {
    return 100;
}
console.log(func());
// Promise {<fulfilled>: 100}
```

### Promise 如何解决回调地狱

> ES7 加入的async/await 的确解决了之前的问题，使开发者在编译过程中更容易理解，语法更清晰并且也不用再单独引用co函数库了

## 总结

异步编程方法|特点
---|---
Generator|生成器函数配合着yield关键词来使用，不自动执行，需要执行next方法一步一步往下执行
Generator+co|通过引入开源co函数库，实现异步编程，并且还能控制返回结果为Promise对象，方便后续继续操作，但是要求yield后面，只能是thunk函数或Promise对象
async/await|ES7 引入的终极异步编程解决方案，不用引入其他任何库，对于await后面的类型无限制，可读性更好，容易理解