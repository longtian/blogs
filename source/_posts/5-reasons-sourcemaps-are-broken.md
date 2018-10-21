---
title: "[译] 导致 SourceMap 无效常见的 4 个原因"
date: 2018-10-21 10:59:16
tags:
categories:
    - 翻译
---

[原文](https://blog.sentry.io/2018/10/18/4-reasons-why-your-source-maps-are-broken)

Souce map 非常好用。换句话说，它们被用来在调试阶段显示源代码，这比线上压缩后的代码好懂多了。从某种意义上讲，source map 可以说是秘密代码（压缩后的代码）的解码器。

但是要让 source map 正常工作可能很棘手。如果你遇到了麻烦，接下来的一些提示或许能帮助你更好的工作。

如果你第一次接触 source map，请在继续阅读前看看这篇早期的博客 [Debugging Minified JavaScript with Source Maps](https://blog.sentry.io/2015/10/29/debuggable-javascript-with-source-maps).

## 丢失或错误的 source map 注释

我们假设你已经通过 [UglifyJS](https://www.npmjs.com/package/uglify-js) 或者 [Webpack](https://webpack.js.org/configuration/devtool/) 生成了一个 source map。但如果只是生成，而浏览器实际上找不到它，那就很划不来了。要做到这一点，浏览器会假设打包好的 JavaScript 文件里有一行含 `sourceMappingURL` 的注释或者返回一个叫 `SourceMap` 的 HTTP 响应头，这个响应头指向 source map 文件的位置。

为了验证 source map 注释能够正常工作，你需要：

**找到文件最后，自成一行的 `sourceMappingURL` 注释**

```js
//# sourceMappingURK=script.min.js.map
```

这个值必须是一个有效的 URI。如果是相对路径，那么它是相对于打包出来的 JavaScript 文件（例如 `script.min.js`）的路径。大多数 source map 生成工具会自动生成这个值，而且提供了选项用于覆盖它。

如果用的是 UglifyJS，可以通过指定 source map 参数 `url=script.min.js.map` 来生成这个注释：

```
# Using UglifyJS 3.3
$ uglifyjs --source-map url=script.min.js.map,includeSources --output script.min.js script.js
```

如果用的是 Webpack ，通过指定 `devtool: "source-map"` 能够开启 source map，Webpack 会在最终生成的文件最后输出 `sourceMappingURL`。你可以通过 `sourceMapFilename` 自定义该文件的名称。

```js
// webpack.config.js
module.exports = {
  // ...
  entry: {
    app: "src/app.js"
  },
  output: {
    path: path.join(__dirname, 'dist'),
    filename: "[name].js",
    sourceMapFilename: "[name].js.map"
  },
  devtool: "source-map"
  // ...
}
```

需要注意的是即使你正确生成了 `sourceMappingURL`，也有可能它没有在最终上线的版本里出现。例如，前端构建工具链里其它的工具可能会移除所有的注释，结果就是把 `//# sourceMappingURL` 也一并删掉。

还有一种情况是你的 CDN 可能会相当智能地把不认识的注释统统删掉；Cloudflare 的自动压缩功能以前就会这么干。所以记住上线后一定要再次确认！

**另外一种做法是：确保服务器返回有效的 SourceMap HTTP 响应头**

除了这个神奇的 `sourceMappingURL` 注释，你还可以通过返回一个 `SourceMap` HTTP 响应头来指定 source map 的地址。

```
SourceMap: /path/to/script.min.js.map
```

和 `sourceMappingURL` 一样，如果这个值是相对路径则相对于打包出来的 JavaScript 文件。浏览器解析 `SourceMap` HTTP 响应头和 `sourceMappingURL` 的规则是一样的。

注意你需要配置你的 web 服务器或者 CDN 来返回这个响应头。但是很多 JavaScript 开发者并不能够随意的修改线上资源的头，所以对大多数人来说，生成 `sourceMappingURL` 要更简单一些。

## 缺少源代码文件

我们假设你已经正确配置好 source map，你的 `sourceMappingURL`（或者 `SourMap` 响应头）存在且生效。到源代码的转换前面部分已经能够正常工作；例如，错误堆栈现在指向源文件的文件名，并且行号和列号也有意义了。尽管这已经算有所提升，但还是缺少一部分，你还是不能通过浏览器的调试工具查看到源代码。

这很有可能是由于你的 source map 文件没有包含或是指向你的源文件导致的。如果没有源文件，你在调试压缩后的代码时还是会卡住。哦天哪。

有几种解决方案可以让源代码文件能够正常工作：

**通过 `sourcesContent` 把源代码嵌到 source map 文件里**

实际上把源代码放到 source map 里是有可能的。在 source map 里，这个字段是 `sourcesContent`。虽然这会导致 source map 的体积增迅速增长(数以兆计)，但是能够非常简单地让浏览器定位并关联你的源文件。如果你为了让浏览器显示源文件而焦头烂额，我们推荐你这么做。

如果你用 UglifyJS，你可以用过 `includeSources` 命令行参数把源代码包含到 source map 的 `sourcesContent` 属性里：

```bash
uglifyjs --source-map url=script.min.js.map,includeSources --output script.min.js script.js
```

如果你用 Webpack，不需要做什么 - Webpack 会默认把源代码包含进 source map （前提是已经打开了 `devtool:"source-map"` 配置）。

**把源代码放到开放服务器上**

除了在 `source map` 里包含源代码，你也可以把它们放到服务器上供浏览器下载。如果你对安全性有担忧，毕竟是你的原始代码，你可以放到 `localhost` 服务器或者确保它们通过 VPN 才能访问（即这些文件只能通过公司内部网络访问）

**Sentry 用户可以上传源文件**

如果你是一个 Sentry 用户并且你的首要目的是确保 source map 文件能够被用来还原堆栈信息以及前后的源代码，你可以试一下第三种方法：使用 sentry-cli 或者直接调用 API [上传源文件](https://docs.sentry.io/platforms/javascript/sourcemaps/#uploading-source-maps-to-sentry)。

当然，如果你用的是前两种方法 - 不管是在 source map 了包含源代码还是放到对外开放的服务器上 - Sentry 都能够找到。这完全取决于你。

## 多次转换导致 source map 失效

如果你用到了两个或以上的 JavaScript 编译器（例如 Babel 和 UglifyJS）独立调用，有可能生成的 source map 文件指向的是一个处于中间转换状态的代码，而不是源代码。这意味着你在浏览器里调试的时候，步进的是未压缩的代码（这已经有所改善）而不是和你的源代码一一对应。

举个例子，你用 Babel 把 ES2018 的代码转换成 ES2015，然后用 UglifyJS 进行压缩

```bash
# Using Babel7.1 and UglifyJS 3.3
$ babel-cli script.js --presets=@babel/env | uglifyjs -o script script.min.js --source -map "filename=app.min.js.map"
$ ls script*
script.js script.min.js script.min.js.map
```

如果你直接用这个命令生成的 source map 文件，你就会发现它并不准确。这是因为这个 source map 只能把压缩后的代码转换成 Bebel 生成的代码。它并不会指向你的源代码。

__注意这个问题在用 Gulp 或者 Grunt 这类任务管理器的时候也很常见。__

要解决这个问题，有两种方案：

**用类似 Webpack 的打包工具管理所有的转换**

不再把 Babel 和 UglifyJS 分开调用，而是用它们的 Webpack 插件形式（例如 [babel-loader](https://github.com/babel/babel-loader) 和 [uglifyjs-webpack-plugin](https://github.com/webpack-contrib/uglifyjs-webpack-plugin)）。Webpack 能够生成单一的 source map 文件来把最终结果转换回源代码，虽然实际上背后依然有多个转换步骤。

**用一个库把不同转换步骤的 source map 串联起来**

如果你决意要分开使用编译器，你可以用 [source-map-merger](https://www.npmjs.com/package/source-map-merger)，或者 Webpack 的 [source-map-loader](https://webpack.js.org/loaders/source-map-loader/) 插件，来把上一步的 source map 吐给下一步的转换。

如果你有的选，还是推荐你用第一步，直接用 Webpack 省得后来哀怨。

## 文件版本不对或缺少版本管理

我们假设你遵循了上面所有的步骤。你的 `sourceMappingURL`（或 `SourceMap` HTTP 响应头）存在并且被正确的声明。你的 source map 包括了你的源代码（或放到公网上）。并且你用了 Webpack 做转换端到端的管理。你的 source map 还是会时不时地映射错误。

还剩下这样的可能：source map 和生成的代码不匹配。

这个问题会在这种情况下会发生：首先、浏览器或者工具下载了一个生成的代码（例如 `script.min.js`），然后试着去下载对应的 source map 文件（`script.min.js.map`），但是下载到的是 “更新” 后的 source map 文件，和之前的生成代码已经不匹配了。

这种情况并不会很常见，但是当你在调试的同时进行部署的时候会发生，或者你调试的是即将过期的、被浏览器缓存的资源时会发生。

**要解决这个问题，你需要管理好文件和 source map 的版本**，有下面几种方式：

* 给每个文件添加版本号，例如：`script.abc123.min.js`
* 在 URL 里添加版本号字符，例如 `script.min.js?abc123`
* 为父级目录添加版本号，例如 `abc123/script.min.js` 

选择哪种策略并不要紧，关键是对所有的 JavaScript 资源要使用一致的策略。最好每一个生成的文件和 source map 都有相同的版本号和命名规则，就像下面这样：

```js
// script.abc123.min.js
for(var a=[i=0];++i<20;a[i]=i);
//# sourceMappingURL=script.abc123.min.js.map
```
用这种方法管理版本能够保证浏览器下载到生成代码和 source map 文件对应上，避免不必要的版本不一致问题。

