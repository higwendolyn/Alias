---
layout: post
title: "Vue源码---自定义指令"
category: 'Vue&React'
---

自定义指令，价值在于当开发人员在某些场景下需要对普通DOM元素进行操作的时候~

## 注册自定义指令

全局注册：通过 ```Vue.directive( id, [definition] )``` 方式注册全局指令

局部注册：通过在```Vue```实例中添加 ```directives``` 对象数据注册局部自定义指令

