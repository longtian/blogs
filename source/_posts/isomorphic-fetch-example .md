---
title: "使用 Fetch API 实现前后端共用同一个 SDK"
date: 2016-01-20 12:00:00
---

[源码](https://github.com/longtian/isomorphic-fetch-example)

架构设计实践: 使用 Fetch API 实现前后端共用同一个 SDK.

做过一段时间的 Web 开发以后,就会有这样的感觉,前后端分离几乎和吃饭睡觉一样平常. 前端专注浏览器的一切,
通过 API 和后端通信,而后端工程师专注于系统的执行效率,可扩展性等问题.

很多优秀的网站特别是有一部分面向开发者的网站都会提供 API 的 SDK.

- 封装底层发起请求的方法
- 异步请求成功和出错的处理

1. 封装

浏览器端发送 AJAX 请求底层一般采用 XMLHttpRequest 来实现, 实际工作中常使用具体的封装良好的类库,
例如 jQuery, superagent 等. 而后端 Node.js 经常用的是基于 Stream 的解决方案, 由于 API 是很常见的
工作,所以已经封装到了 Node.JS 的核心模块 http 里, 此外比较常用的第三方的类库有 request.

如果前后端公用一个 SDK, 那么必然要在上层引入一定的设计模式,来封装底层具体实现:

```js
if(isBrowser){
    return facade(new XMLHttpRequest(...))
}else{
    return facade(http.request(...))
}
```

在浏览器端发起 AJAX 请求现在有了一套新的API, 那就是 Fetch. 与基于 XMLHttpRequest 的方法相比,它有几个显著地优势.

- 更安全

虽然 Fetch 目前只在 Chrome 等比较新的浏览器上可以使用. 不过别担心, 已经有好几个垫片可以解决兼容性的问题.

2. 异步

不管是在客户端还是在服务器端,对请求的处理都是异步的,这是由 JavaScript 本身的特性决定的.在很长的一段时间里
回调函数一直都是处理异步问题首选的方案,但是在经过了这么多年的折磨之后,人们终于想到了一个非常优秀的替代方案 - Promise.

```js
var api=require('my-awesome-api');
api.repos('github')
    .then(res=>{
        // 成功
    })
    .catch(err=>{
        // 失败
    })
```

如果在客户端和在服务器端能够采用一致的方案,这无疑会大大提高 JavaScript 代码的质量和开发效率.

公用 SDK 有很多好处:

- 浏览器端专注于路由,页面的设计,把请求的封装处理交给 SDK
- 后端的开发者可以使用 SDK

所以是时候尝试和采用 Fetch API 了.

### 关于 Fetch API 简单的介绍

http://github.github.io/fetch/

### 实现该模式的 SDK

https://github.com/ringcentral/ringcentral-js

### 关键模块

https://github.com/matthew-andrews/isomorphic-fetch
