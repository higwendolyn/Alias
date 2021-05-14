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
* **binding**: 一个对象，包含以下 property：
    + ```name```：指令名，不包括 ```v-``` 前缀。
    + ```value```：指令的绑定值，例如：```v-my-directive="1 + 1"``` 中，绑定值为 2。
    + ```oldValue```：指令绑定的前一个值，仅在 ```update``` 和 ```componentUpdated``` 钩子中可用。无论值是否改变都可用。
    + ```expression```：字符串形式的指令表达式。例如 ```v-my-directive="1 + 1"``` 中，表达式为 ```"1 + 1"```。
    + ```arg```：传给指令的参数，可选。例如 ```v-my-directive:foo``` 中，参数为 ```"foo"```。
    + ```modifiers```：一个包含修饰符的对象。例如：```v-my-directive.foo.bar``` 中，修饰符对象为 ```{ foo: true, bar: true }```。
* **vnode**：Vue 编译生成的虚拟节点。
* **oldVnode**：上一个虚拟节点，仅在 ```update``` 和 ```componentUpdated``` 钩子中可用。

> 除了 ```el``` 之外，其它参数都应该是只读的，切勿进行修改。如果需要在钩子之间共享数据，建议通过元素的 ```dataset``` 来进行。

```html
<div v-demo="{ color: 'white', text: 'hello!' }"></div>
<script>
    Vue.directive('demo', function (el, binding) {
    console.log(binding.value.color) // "white"
    console.log(binding.value.text)  // "hello!"
    })
</script>
```

### 动态指令参数

指令的参数可以是动态的

> 例如，在 ```v-mydirective:[argument]="value"``` 中，```argument``` 参数可以根据组件实例数据进行更新

例子
```html
<div id="dynamicexample">
  <h3>Scroll down inside this section ↓</h3>
  <p v-pin:[direction]="200">I am pinned onto the page at 200px to the left.</p>
</div>
Vue.directive('pin', {
  bind: function (el, binding, vnode) {
    el.style.position = 'fixed'
    var s = (binding.arg == 'left' ? 'left' : 'top')
    el.style[s] = binding.value + 'px'
  }
})
```

```javascript
new Vue({
  el: '#dynamicexample',
  data: function () {
    return {
      direction: 'left'
    }
  }
})
```

## 实践

先显示默认背景图，当图片资源真正加载出来了之后，再把真实图片放置到对应的位置上并显示出来

```html
<div id="app2" class="demo">
    <div v-for="item in imageList">
        <img src="../assets/image/bg.png" alt="默认图" v-image="item.url">
    </div>
</div>
<script>
    Vue.directive("image", {
        inserted: function(el, binding) {
            setTimeout(function(){
                el.setAttribute("src", binding.value);
            }, Math.random() * 1200)
        }
    })
    new Vue({
        el: "#app2",
        data: {
            imageList: [
                {
                    url: "http://consumer-img.waixing.com/content/dam/waixing-cbg-site/greate-china/cn/mkt/homepage/section4/home-s4-p10-plus.jpg"
                },
                {
                    url: "http://consumer-img.waixing.com/content/dam/waixing-cbg-site/greate-china/cn/mkt/homepage/section4/home-s4-watch2-pro-banner.jpg"
                },
                {
                    url: "http://consumer-img.waixing.com/content/dam/waixing-cbg-site/en/mkt/homepage/section4/home-s4-matebook-x.jpg"
                }
            ]
        }
    })
</script>
```