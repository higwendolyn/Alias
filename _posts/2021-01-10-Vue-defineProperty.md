---
layout: post
title: "Vue源码---响应式原理（未完）"
category: 'Vue'
---

Vue源码之响应式原理学习，包括vue2.x版本和vue3.0。

![image.png](../../../images/defineProperty1.png)

## 观察者模式

观察者模式，它定义了一种 **一对多** 的关系，让多个观察者对象同时监听某一个主题对象，这个主题对象的状态发生变化时就会通知所有的观察者对象，使得它们能够自动更新自己。在观察者模式中有两个主要角色：Subject（主题）和 Observer（观察者）。

![image.png](../../../images/defineProperty2.png)

由于观察者模式支持简单的广播通信，当消息更新时，会自动通知所有的观察者。

使用 TypeScript 来实现观察者模式:
### ConcreteObserver

```typescript
interface Observer {
  notify: Function;
}

class ConcreteObserver implements Observer{
  constructor(private name: string) {}
  notify() {
    console.log(`${this.name} has been notified.`);
  }
}
```

###  Subject 类

```typescript
class Subject { 
  private observers: Observer[] = [];

  public addObserver(observer: Observer): void {
    this.observers.push(observer);
  }

  public notifyObservers(): void {
    console.log("notify all the observers");
    this.observers.forEach(observer => observer.notify());
  }
}
```

### 使用示例

```typescript
// ① 创建主题对象
const subject: Subject = new Subject();

// ② 添加观察者
const observerA = new ConcreteObserver("ObserverA");
const observerC = new ConcreteObserver("ObserverC");
subject.addObserver(observerA); 
subject.addObserver(observerC);

// ③ 通知所有观察者
subject.notifyObservers();
```

主要包含三个步骤：

① 创建主题对象、② 添加观察者、③ 通知观察者。

上述代码成功运行后，控制台会输出以下结果：

```
notify all the observers
ObserverA has been notified.
ObserverC has been notified.
```

 这三个步骤实现自动化，这就是实现响应式的核心思路。

 ![image.png](../../../images/defineProperty3.png)

## Vue响应式原理(2.6.12)

* 从 Vue 初始化，到首次渲染生成 DOM 的流程
* 从 Vue 数据修改，到页面更新 DOM 的流程

### Vue 初始化

```
<template>
  <div>
    {{ message }}
  </div>
</template>
<script>
new Vue({
  data() {
    return {
      message: "hello kitty",
    };
  },
});
</script>
```

new Vue 开始

```javascript
// 执行 new Vue 时会依次执行以下方法
// 1. Vue.prototype._init(option)
// 2. initState(vm)
// 3. observe(vm._data)
// 4. new Observer(data)

// 5. 调用 walk 方法，遍历 data 中的每一个属性，监听数据的变化。
function walk(obj) {
  const keys = Object.keys(obj);
  for (let i = 0; i < keys.length; i++) {
    defineReactive(obj, keys[i]);
  }
}

// 6. 执行 defineProperty 监听数据读取和设置。
function defineReactive(obj, key, val) {
  // 为每个属性创建 Dep（依赖搜集的容器，后文会讲）
  const dep = new Dep();
  // 绑定 get、set
  Object.defineProperty(obj, key, {
    get() {
      const value = val;
      // 如果有 target 标识，则进行依赖搜集
      if (Dep.target) {
        dep.depend();
      }
      return value;
    },
    set(newVal) {
      val = newVal;
      // 修改数据时，通知页面重新渲染
      dep.notify();
    },
  });
}
```

数据描述符绑定完成后：

![image.png](../../../images/defineProperty4.png)

Vue 初始化时，进行了数据的 get、set 绑定，并创建了一个 Dep 对象

Dep 对象用于依赖收集，它实现了一个发布订阅模式，完成了数据 Data 和渲染视图 Watcher 的订阅

```typescript
class Dep {
  // 根据 ts 类型提示，我们可以得出 Dep.target 是一个 Watcher 类型。
  static target: ?Watcher;
  // subs 存放搜集到的 Watcher 对象集合
  subs: Array<Watcher>;
  constructor() {
    this.subs = [];
  }
  addSub(sub: Watcher) {
    // 搜集所有使用到这个 data 的 Watcher 对象。
    this.subs.push(sub);
  }
  depend() {
    if (Dep.target) {
      // 搜集依赖，最终会调用上面的 addSub 方法
      Dep.target.addDep(this);
    }
  }
  notify() {
    const subs = this.subs.slice();
    for (let i = 0, l = subs.length; i < l; i++) {
      // 调用对应的 Watcher，更新视图
      subs[i].update();
    }
  }
}
```
对 Dep 的源码分析:

![image.png](../../../images/defineProperty5.png)

了解 Data 和 Dep 之后，我们讲一下 Watcher

```typescript
class Watcher {
  constructor(vm: Component, expOrFn: string | Function) {
    // 将 vm._render 方法赋值给 getter。
    // 这里的 expOrFn 其实就是 vm._render，后文会讲到。
    this.getter = expOrFn;
    this.value = this.get();
  }
  get() {
    // 给 Dep.target 赋值为当前 Watcher 对象
    Dep.target = this;
    // this.getter 其实就是 vm._render
    // vm._render 用来生成虚拟 dom、执行 dom-diff、更新真实 dom。
    const value = this.getter.call(this.vm, this.vm);
    return value;
  }
  addDep(dep: Dep) {
    // 将当前的 Watcher 添加到 Dep 收集池中
    dep.addSub(this);
  }
  update() {
    // 开启异步队列，批量更新 Watcher
    queueWatcher(this);
  }
  run() {
    // 和初始化一样，会调用 get 方法，更新视图
    const value = this.get();
  }
}
```

源码中我们看到，Watcher 实现了渲染方法 ```_render``` 和 Dep 的关联， 初始化 Watcher 的时候，打上 Dep.target 标识，然后调用 get 方法进行页面渲染。

加上上文的 Data，目前 Data、Dep、Watcher 三者的关系如下：

![image.png](../../../images/defineProperty6.png)

整体流程：

Vue 通过 ```defineProperty``` 完成了 Data 中所有数据的代理，当数据触发 get 查询时，会将当前的 Watcher 对象加入到依赖收集池 Dep 中，当数据 Data 变化时，会触发 set 通知所有使用到这个 Data 的 Watcher 对象去 update 视图。

![image.png](../../../images/defineProperty7.png)

上图的流程中 Data 和 Dep 都是 Vue 初始化时创建的，但现在我们并不知道 Wacher 是从哪里创建的