---
layout: post
title: "JS基础---Process.nextTick原理"
category: 'JavaScript系列文章'
---

理解Process.nextTick原理，深入理解Vue的nextTick~

## 异步更新

**例1**：

```html
<div id="app">
    <div class="msg">
        {{msg}}
    </div>
</div>
new Vue({
    el: '#app',
    data: function(){
        return {
            msg: ''
        }
    },
    mounted(){
        this.msg = '我是测试文字'
        console.log(document.querySelector('.msg').offsetHeight) //0
    }
})
```

这时不管怎么获取，文字的Div高度都是0；但是直接获取却是有值

![image.png](../../../images/nextTick1.png)

---

**例2**：

``` javascript
export default {
  data () {
    return {
      msg: 0
    }
  },
  mounted () {
    this.msg = 1
    this.msg = 2
    this.msg = 3
  },
  watch: {
    msg () {
      console.log(this.msg)
    }
  }
}
```

* 猜测：依次打印：1、2、3；
* 实际：只会输出一次：3。

### Vue 异步更新

上述两个问题的发生，都是在给data中赋值后立马去查看数据导致的。由于“查看数据”这个动作是同步操作的，而且都是在赋值之后；因此给数据赋值操作是一个异步操作。

Vue官网对数据操作描述：

> 可能你还没有注意到，Vue 在更新 DOM 时是异步执行的。只要侦听到数据变化，Vue 将开启一个队列，并缓冲在同一事件循环中发生的所有数据变更。如果同一个 watcher 被多次触发，只会被推入到队列中一次。这种在缓冲时去除重复数据对于避免不必要的计算和 DOM 操作是非常重要的。然后，在下一个的事件循环“tick”中，Vue 刷新队列并执行实际 (已去重的) 工作。Vue 在内部对异步队列尝试使用原生的 Promise.then、MutationObserver 和 setImmediate，如果执行环境不支持，则会采用 setTimeout(fn, 0) 代替。


