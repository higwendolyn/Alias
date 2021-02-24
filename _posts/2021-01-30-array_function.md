---
layout: post
title: "JS基础---手写数组多个方法的底层实现"
category: 'JavaScript'
---

比较常用的数组方法有push、pop、slice、map和reduce等。

问题1：
<font style="color: #ec7907;">reduce 方法里面的参数都是什么作用？</font>

问题2：
<font style="color: #ec7907;">push 和 pop 的底层逻辑是什么样的呢？</font>

## push 方法的底层实现

```javascript
Array.prototype.push = function(...items) {
    let O = Object(this); // ecma 中提到的先转换为对象
    let len = this.length >>> 0;
    let argCount = items.length >>> 0;
    // 2 ^ 53 - 1为JS能表示的最大正整数
    if (len + argCount > 2 ** 53 - 1) {
        throw new TypeError('The number of array is over the max value')
    }
    for (let i = 0; i < argCount; i++) {
        O[len + i] = items[i];
    }
    let newLength = len + argCount;
    O.length = newLength;
    return newLength;
}
```

<font style="color: red">为什么一些变量要进行无符号位移？</font>

## pop 方法的底层实现

```javascript
Array.prototype.pop = function() {
    let O = Object(this);
    let len = this.length >>> 0;
    if (len === 0) {
        O.length = 0;
        return undefined;
    }
    len--;
    let value = O[len];
    delete O[len];
    O.length = len;
    return value;
}
```

## map 方法的底层实现

```javascript
Array.prototype.map = function(callbackFn, thisArg) {
    if (this === null || this === undefined) {
        throw new TypeError('Cannot read property 'map' of null');
    }
    if (Object.prototype.toString.call(callbackFn) != '[object Function]') {
        throw new TypeError(callbackFn + 'is not a function');
    }
    let O = Object(this);
    let T = thisArg;

    let len = O.length >>> 0;
    let A = new Array(len);
    for (let k = 0; k < len; k++) {
        if (k in O) {
            let kValue = O[k];
            // 依次传入this，当前项，当前索引，整个数组
            let mappedValue = callbackFn.call(T, kValue, k, O);
            A[k] = mappedValue;
        }
    }
    return A;
}
```