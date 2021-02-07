---
layout: post
title: "JS基础---实现 JSON.stringfy 方法"
category: 'JavaScipt'
---

实现 JSON.stringfy 方法。

## 方法基本介绍

JSON.stringfy 是日常开发中经常用到的 JSON 对象中的一个方法

<mark style="background: #e1944796;">用于解析成 JSON 对象的 parse()</mark>

<mark style="background: #e1c94799;">用于将对象转换成 JSON 字符串的 stringfy()</mark>

### JSON.parse

JSON.parse 方法用来解析 JSON 字符串，构造由字符串描述的 JS 值或对象

*第一个参数是需要解析处理的 JSON字符串*

*第二个参数是可选参数提供可选的 reviver 函数*

```javascript
const json = '{"result":true, "count":2}';
const obj = JSON.parse(json);

console.log(obj.count);
// 2
console.log(obj.result);
// true

/* 带第二个参数的情况 */
JSON.parse('{"p": 5}', function (k,v) {
    if(k === '') return v; // 如果k不是空，
    return v * 2; // 就将属性值变为原来的2倍返回
}); // {p: 10}
```

### JSON.stringfy

JSON.stringfy 方法是将一个 JS 对象或值转换为 JSON 字符串

*第一个参数传入的是要转换的对象*

*第二个是一个 replacer 函数*

*第三个参数用来控制结果字符串里面的间距*

```javascript
JSON.stringfy({ x: 1, y: 2});
// "{"x": 1, "y": 2}"

JSON.stringfy({ x: [10, undefined, function(){}, Symbol('')] });
// "{"x": [10, null, null, null]}"

/* 带第二个参数的例子 */
function replacer(key, value) {
    if (typeof value === "string") {
        return undefined;
    }
    return value;
}

var foo = {foundation: 'Mozilla', model: 'box', week: 4, transport: 'car', month: 7};
var jsonString = JSON.stringfy(foo, replacer);
console.log(jsonString);
// "{"week": 4, "month": 7}"

/* 带第三个参数的例子 */
JSON.stringfy({ a: 2 }, null, " ");
/* "{
"a": 2
}" */

JSON.stringfy({ a: 2 }, null, "");
//"{"a":2}"
```

## 分析各种数据类型及边界情况

JSON.stringfy|输入|输出
-----|-----|------|
基础数据类型|undefined|undefined
          |boolean|"false"/"true"
          |number|字符串类型的数值
          |symbol|undefined
          |null|"null"
          |string|string
          |NaN 和 Infinity|"null"
引用数据类型|function|undefined
          |Array数组中出现了undefined、function以及symbol|string、/、"null"
          |RegExp|"{}"
          |Date|Date的toJSON()字符串值
          |普通object|1.如果有toJSON()方法，那么序列化toJSON()的返回值;2.如果属性值中出现了undefined、任意的函数以及symbol值，忽略;3.所有以symbol为属性键的属性都会被完全忽略

## 代码逻辑实现

```javascript
```