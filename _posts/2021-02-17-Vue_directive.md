---
layout: post
title: "Vue源码---自定义指令"
category: 'Vue&React'
---

自定义指令，价值在于当开发人员在某些场景下需要对普通DOM元素进行操作的时候~

## 注册自定义指令

* 全局注册：通过 ```Vue.directive( id, [definition] )``` 方式注册全局指令

* 局部注册：通过在```Vue```实例中添加 ```directives``` 对象数据注册局部自定义指令


```html
<div id="app" class="demo">
    <!-- 全局注册 -->
    <input type="text" placeholder="我是全局自定义指令" v-focus>
</div>
<script>
    Vue.directive("focus", {
        inserted: function(el){
            el.focus();
        }
    })
    new Vue({
        el: "#app"
    })
</script>
```

```html
<div id="app" class="demo">
    <!-- 局部注册 -->
    <input type="text" placeholder="我是局部自定义指令" v-focus2>
</div>
<script>
    new Vue({
        el: "#app",
        directives: {
            focus2: {
                inserted: function(el){
                    el.focus();
                }
            }
        }
    })
</script>
```


## 钩子函数

### 钩子函数

一个指令定义对象的几个钩子函数 (均为可选)：

* ```bind```：只调用一次，指令第一次绑定到元素时调用。
> 在这里可以进行一次性的初始化设置。

* ```inserted```：被绑定元素插入父节点时调用 (仅保证父节点存在，但不一定已被插入文档中)。
* ```update```：所在组件的 VNode 更新时调用。
* ```componentUpdated```：指令所在组件的 VNode **及其子 VNode** 全部更新后调用。
* ```unbind```：只调用一次，指令与元素解绑时调用。

### 钩子函数传入参数

指令钩子函数会被传入以下参数:

* **el**: 指令所绑定的元素，可以用来直接操作 DOM，就是放置指令的那个元素。
* **binding**: 一个对象，里面包含了几个属性，这里不多展开说明，官方文档上都有很详细的描述。
* **vnode**：Vue 编译生成的虚拟节点。
* **oldVnode**：上一个虚拟节点，仅在 update 和 componentUpdated 钩子中可用。
