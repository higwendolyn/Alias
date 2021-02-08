---
layout: post
title: "Cookie、Session和Web Storage（未完）"
category: '网络'
---

重点解析的Cookie、Session和Web Storage区别与使用。

## Cookie 和 Session

**会话**可简单理解为：用户开一个浏览器，点击多个超链接，访问服务器多个web资源，然后关闭浏览器，整个过程称之为一个会话。

在会话中，需要保存一些数据，如登录信息、访问购物网站的购物信息等，所以出现了两种 **保存会话信息** 的技术：cookie 和 session。

## Cookie

### 什么是Cookie

Cookie是客户端技术，程序把每个用户的数据以cookie的形式写给用户各自的浏览器。当用户使用浏览器再去访问服务器中的web资源时，就会带着各自的数据去。这样，web资源处理的就是用户各自的数据了。

<font style="color: red;">Cookie技术通过请求和响应报文中写入Cookie信息来控制客户端的状态</font>

第一次发起请求

![image.png](../../../images/cookie1.png)

客户端保存了Cookie之后的发起请求

![image.png](../../../images/cookie2.png)

首部字段内没有Cookie的相关信息，能看到set-Cookie里的信息，这就是服务器端生成的Cookei信息。

![image.png](../../../images/cookie3.png)

请求报文里都自动发送Cookie信息

![image.png](../../../images/cookie4.png)

### Cookie细节

* 一个Cookie只能标识一种信息，它至少含有一个标识该信息的<mark>名称</mark>（NAME）和<mark>设置值</mark>（VALUE）。
* 一个WEB站点可以给一个WEB浏览器发送多个Cookie，一个WEB浏览器也可以存储多个WEB站点提供的Cookie。
* 浏览器一般只允许存放300个Cookie，每个站点最多存放20个Cookie，每个Cookie的大小限制为<mark>4KB</mark>。
* 如果创建了一个cookie，并将他发送到浏览器，默认情况下它是一个会话级别的cookie（即存储在浏览器的内存中），用户退出浏览器之后即被删除。若希望浏览器将该cookie存储在磁盘上，则需要使用Max-Age，并给出一个以秒为单位的时间。<mark>将最大时效设为0则是命令浏览器删除该cookie</mark>。
* 注意，删除cookie时，<mark>path必须一致</mark>，否则不会删除

### set-Cookie的字段的属性

```
Set-Cookie: logcookie=3qjj; expires=Wed, 13-Mar-2019 12:08:53 GMT; Max-Age=31536000; path=/;
 domain=fafa.com;secure; HttpOnly;
```
1. ```logcookie=3qjj``` 赋予Cookie的**名称**和**值**，logcookie是名字 ，3qjj是值
2. **expires** 是设置cookie有效期。
>当省略expires属性时，Cookie仅在关闭浏览器之前有效。可以通过覆盖已过期的Cookie，设置这个Cookie的过期时间是过去的时间，实现对客户端Cookie 的实质性删除操作。
3. **path** 是限制指定Cookie 的发送范围的文件目录。
>不过另有办法可避开这项限制，看来对其作为安全机制的效果不能抱有期待。
4. **domain** 通过domain属性指定的域名可以做到与结尾匹配一致。
>比如，指定domain是fafa.com，除了fafa.com那么www.fafa.com等都可以发送Cookie。
5. **secure** 设置web页面只有在HTTPS安全连接时，才可以发送Cookie。HTTP则不可以进行。
6. **HttpOnly** 它使JavaScript 脚本无法获得Cookie。
>通过上述设置，通常从Web 页面内还可以对Cookie 进行读取操作。但使用JavaScript 的document.cookie 就无法读取Cookie 的内容了


**优点**：
* 简单
* 不需要服务器资源
* 可以配置 cookie 的过期日期，数据持久性存储

**缺点**：
* cookie 的数量和长度有限制
* 同源请求时携带传递，损耗带宽（cookie 信息越大，完成对服务器请求的时间越长）
* 安全性（cookie 数据并非存储在安全环境，可以被他人访问）

### cookie和session的区别

* cookie数据存放在客户的浏览器上，**session数据放在服务器上**
* cookie不是很安全，别人可以分析存放在本地的cookie并进行cookie欺骗，**考虑到安全应当使用session。用户验证这种场合一般会用 session**
* **session保存在服务器，客户端不知道其中的信息**；反之，cookie保存在客户端，服务器能够知道其中的信息
* **session会在一定时间内保存在服务器上，当访问增多，会比较占用你服务器的性能，考虑到减轻服务器性能方面，应当使用cookie**
* **session中保存的是对象**，cookie中保存的是字符串
* **session不能区分路径**，同一个用户在访问一个网站期间，所有的session在任何一个地方都可以访问到，而cookie中如果设置了路径参数，那么同一个网站中不同路径下的cookie互相是访问不到的
