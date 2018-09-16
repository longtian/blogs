---
title: 基于 Webpack 的跨平台开发
date: 2018-09-16 09:07:50
tags:
---

小程序、快应用的开发最近相当热门，公司也在开发对应的 SDK 。既然小程序、快应用都是选用的 JavaScript 做为开发语言，那么有没有
可能，让小程序和快应用都能公用基于 H5 的 SDK 核心。

答案是肯定可以！如果用一个词来概括软件工程最想解决的问题，那么 `问题拆分` 也许是最合适的，我们把问题拆解：

- JavaScript 核心
- 对不同平台接口的抽象

`JavaScript 核心` 不用说，和具体的业务逻辑有关，设计之初就要清晰的知道系统的边界在哪里，核心和接口之间如何通信。

主要的问题在如何降低接口抽象的复杂度。

## 环境变量 - 代码如何知道当前运行的环境

现代软件开发越来越重视过程，例如很多软件都很明显的区分为 `构建` 和 `运行` 两个部分。通过参数的不同取值指定运行环境
是 `构建` 过程一个比较常用的步骤，这样做的好处是能够为不同平台构建出不同的工件，有利于减小工件的体积。

```bash
# 为 H5 打包
PLATFORM=H5 build
```

```bash
# 为快应用打包
PLATFORM=quickapp build
```

这样就通过环境变量把运行时的环境传给了负责构件的命令 `build` (这里假设构件的命令是 build)。

下面的代码就能够打出不同的包，并在不同平台上会打印出不同的内容：

```js
// app.js
console.log(`我在 ${process.env.PLATFORM} 平台下`);
```

**不同平台如何访问环境变量**

`process` 其实是 Node.js 全局定义的一个属性，那么为什么在非 Node.js 平台下也能访问到呢？这就要借助 **<del>前端</del> JavaScript 打包** 
工具 `webpack` 了, 关于 process 的处理有两种情况：

- 在全局会有一个 mock 的 process 对象，确保相关代码能够访问到 process 对象而不会报错
- 通过 `DefinePlugin` 替换代码里的 `process.env.PLATFORM`

## 模块隔离 - 根据平台执行不同的逻辑 

知道了当前的运行环境，就能够根据平台执行不同的逻辑了

**一般做法**

其实就是 `if else` ，代码里一定多多少少有一些判断当前平台的代码。

```js
// app.js
if(process.env.PLATFORM === 'H5'){
 // 如果是 H5 平台就执行这里的代码
}
```

通过 `if else` 虽然能够解决问题，但是当代码比较复杂的时候还是会导致难以维护，比如下面这种情况：

```js
// app.js
if(process.env.PLATFORM === 'H5'){
  // H5 相关逻辑
}else if(process.env.PLATFORM === 'quickapp'){
  // 快应用下要加载一个模块
  const dep = require('some-dependency');
}else{
  // 其它
}
```

因此这种做法只适合代码比较简单的情况，例如传递一些 `flag`，否则可以用下面这种方法。

**更优方案**

假设 H5、快应用、小程序都需要用到一个模块叫做 `foo`，引入的源代码如下：

```js
// app.js
require('./foo')
```

目录结构如下，`foo` 模块对应不同平台有不同实现，但是暴露的方法签名以及返回类型是一致的：

```
.
foo.js
foo.quickapp.js
foo.miniprogram.js
```

通过 `webpack` 的 `config` 参数可以指定不同平台的配置文件。

| 平台          | webpack 配置文件               |
|--------------|-------------------------------|
| h5           | webpack.config.js             |
| miniprogram  | webpack.config.miniprogram.js |
| quickapp     | webpack.config.quickapp.js    |
 
```js
// webpack.config.js
module.exports = {
  //...
  resolve: {
    extensions: ['.js', '.json']
  }
};
```

如果是快应用的话就修改成这样

```js
// webpack.config.quickapp.js
module.exports = {
  //...
  resolve: {
    extensions: ['.quickapp.js', '.js', '.json']
  }
};
```

可以看到 `resolve.extensions` 其实是一个数组，当 `webpack` 遇到 `require` 一个文件依赖的时候会按照这个顺序进行匹配。

## 命名空间 - 解决第三方模块依赖于浏览器的问题

这其实是开发过程过程中一个非常具体的问题，这个模块是 [jsencrypt](https://www.npmjs.com/package/jsencrypt)。

在 `H5` 平台下引入了这个模块，打包后运行没有问题，但是当尝试在快应用上运行的时候一直报这个错：

```js
"window is undefined"
```

显然这个插件是为了 H5 开发的，没有考虑其它平台的兼容性。

**一般做法**

直接把源码下下来丢到 `vendors` 文件夹下，然后把 `window`、`navigator` 相关的兼容性一个一个问题修复。这种做法其实放弃了使用 `npm` 管理依赖的优势，
并且在项目中留下了一个技术债。以后如果 `jsencrypt` 发布了一个新的不得不使用的版本（例如修复某个安全漏洞），需要手动更新依赖并且打补丁。

**更优方案**

后来突然想起来网上有人遇到过类似的情况：社区有很多的 jQuery 插件，这些插件在编写的时候会假设全局有一个 $ 对象，但是使用了 `webpack` 以后，
由于用到了闭包，全局环境下其实是没有 `$` 这个对象的的，而解决这个问题一个比较通用的做法是使用 `webpack` 的 `ProvidePlugin`。

```js
// webpack.config.js
module.exports = {
  //...
  plugins: [
    //...,
    new webpack.ProvidePlugin({
      $: 'jquery'
    })
  ]
};
```

顺着这个思路，修改了一下 `webpack` 配置文件：

```js
// webpack.config.quickapp.js
module.exports = {
  //...
  plugins: [
    //...,
    new webpack.ProvidePlugin({
      window: path.join(__dirname, 'noop.js'),
      navigator: path.join(__dirname, 'noop.js')
    })
  ]
};
```

而 `noop` 的代码也非常简单，就是返回一个空对象

```js
// noop.js
module.exports = {};
```

这样至少可以保证所有第三方模块都不会报 `window` 或者 `navigator` 找不到的错误。

### 总结

通过上面几个由浅入深的问题，我们都够了解到 `webpack` 不仅仅是前端的打包工具，也可以用于跨平台的开发。

| 解决问题   | Webpack 配置        | 试用场景                            |
|-----------|--------------------|------------------------------------|
| 环境变量   | DefinePlugin       | 判断平台，传递 Flag 等较为简单的逻辑     |
| 模块隔离   | resolve.extensions | 引入同一个模块在不同平台下的实现         |
| 命名空间   | ProvidePlugin      | 修复兼容性问题，提供跨平台的全局空间     |
