---
layout: post
title: "web安全-—XSS 和 CSRF"
category: '浏览器'
---

在 Web 安全领域中，XSS 和 CSRF 是最常见的攻击方式。本文将会简单介绍 XSS 和 CSRF 的攻防问题。

## XSS 攻击

### 什么是 XSS
Cross-Site Scripting（跨站脚本攻击）简称 XSS，是一种代码注入攻击。

> 攻击者通过在目标网站上注入恶意脚本，使之在用户的浏览器上运行。利用这些恶意脚本，攻击者可获取用户的敏感信息如 Cookie、SessionID 等，进而危害数据安全。
>
> 为了和 CSS 区分，这里把攻击的第一个字母改成了 X，于是叫做 XSS。

<mark>XSS 的本质是</mark>：恶意代码未经过滤，与网站正常的代码混在一起；浏览器无法分辨哪些脚本是可信的，导致恶意脚本被执行。

用户是通过哪种方法“注入”恶意脚本的呢？

* 来自用户的 UGC 信息
* 来自第三方的链接
* URL 参数
* POST 参数
* Referer （可能来自不可信的来源）
* Cookie （可能来自其他子域注入）

XSS 分类
根据攻击的来源，XSS 攻击可分为存储型、反射型和 DOM 型三种。

|类型|存储区|插入点|
|存储型 XSS|后端数据库|HTML| 
|反射型 XSS|URL|HTML|
|DOM 型 XSS|后端数据库/前端存储/URL|前端 JavaScript|

存储区：恶意代码存放的位置。
插入点：由谁取得恶意代码，并插入到网页上。

## 参考文章

[浅说 XSS 和 CSRF](https://github.com/dwqs/blog/issues/68)

[前端安全系列（一）：如何防止XSS攻击？](https://tech.meituan.com/2018/09/27/fe-security.html)

[前端安全系列（二）：如何防止CSRF攻击？](https://tech.meituan.com/2018/10/11/fe-security-csrf.html)