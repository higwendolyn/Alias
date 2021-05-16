---
layout: post
title: "Vue-—keep-alive（未完）"
category: 'Vue&React'
---

## keep-alive 用法

```html
<keep-alive>
  <component :is="view"></component>
</keep-alive>
```

* props:
    + include：只有名称匹配的组件才会被缓存
    + exclude: 任何名称匹配的组件都不会被缓存
    + max: 最多可以缓存多少组件实例。(2.5.0 新增, 一旦这个数字达到了，在新实例被创建之前，已缓存组件中最久没有被访问的实例会被销毁掉)

* 用法
    + keep-alive 包裹动态组件时，会缓存不活动的组件实例，而不是销毁他们。
    + 当组件在  keep-alive 内被切换， 它的 activated 和 deactivated 两个生命周期钩子函数将会被执行。

## 实现原理


LRU缓存策略
