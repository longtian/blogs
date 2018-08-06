---
title: "被遗忘的 Loader"
date: 2016-05-03 12:00:00
---

[源码](https://github.com/longtian/introduction-to-statsd)

> 专门介绍 Webpack 那些冷门的 Loader

Webpack 的官方文档上有一个 [list-of-loaders](http://webpack.github.io/docs/list-of-loaders.html)
之是其中的大部分被使用到的机会非常少, 好心疼这些缺 :heart: 的模块, 所以有这个 repo 专门介绍这些冷门 Loader .

[demo](https://wyvernnot.github.io/other-webpack-loaders)

- [json](#json-loader)
- [script](#script-loader)
- 等等

# raw-loader

作用:以 utf-8 编码加载文件的内容


# json-loader

作用:加载一个JSON文件的内容并返回对象.

在 Node.JS 环境下可以 require 一个 .json 格式的文件:

```
require('./config.json')
```

但是在 wepack 打包的时候不可以直接 require 一个 .json 的文件. 这是因为 webpack 会把 .json 文件当成是 .js 文件来解析导致报错.
因此正确的做法是使用 json-loader

```
require('json!./config.json')
```

# hson-loader

作用:加载增强版的JSON (HSON) 并返回对象.

[HSON](https://github.com/timjansen/hanson) 对 JSON 的增强体现在:

- 所有的 JSON 字符串都是有效的 HSON 字符串
- HSON 里可以写注释
- HSON 里属性名不一定要有引号

# script-loader

有的时候不希望 webpack 在打包某些文件的时候进行解析, 而是希望在浏览器中直接运行.
一个不常见的例子是某段代码重新定义了 require , 如下:

**browser_require.js**

```
window.require=function(){
    console.log(arguments);
}
require(2);
```

引入该文件的正确方法是:

```
require('script!./browser_require.js')
```

这样该文件内容里的 require 就不会被 webpack 解析了, 使用 script-loader 整个文件会被转成字符串打到包里,最终在浏览器里通过 eval 来执行.
