---
layout: post
title: "JS基础---实现一个EventEmitter"
category: 'JavaScript系列文章'
---

进入 Node.js 的 events 模块以及 EventEmitter 的学习，并且实现它的底层逻辑。

> events 模块属于 Node.js 服务端的知识，但是由于大多数 Node.js 核心API 构建用的是异步时间驱动架构

问题1：
<font style="color: #ec7907;">EventEmitter 采用的是什么设计模式？</font>

问题2：
<font style="color: #ec7907;">EventEmitter 常用的API是怎样实现的？</font>

