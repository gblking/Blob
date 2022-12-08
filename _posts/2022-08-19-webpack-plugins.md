---
layout: post
title: webpack plugins
categories: webpack
tags: [webpack, plugins]
excerpt: webpack配置plugins，以及常用的plugin
---

## 简介

webpack 有着丰富的插件接口，插件接口使得 webpack 变得更加极其灵活。  
可以在`webpack.config.js`、`vue.config.js`中进行 plugins 配置, 示例：

```js
// webpack.config.js
const HtmlWbpackPlugin = require("html-webpack-plugin");
module.exports = {
  ...
  plugins: [new HtmlWbpackPlugin()]
}
```

```js
// vue.config.js
const HtmlWbpackPlugin = require("html-webpack-plugin");
module.exports = {
  ...
  configureWebpack: {
    plugins: [new HtmlWbpackPlugin()]
  }
}
```

有些插件是 webpack 自带的，有些属于第三方插件

## 插件收集

### HtmlWebpackPlugin

[插件文档](https://github.com/jantimon/html-webpack-plugin#configuration)

创建 html 文件，用于服务器访问。以 vue 为例，项目中都有个 index.html 文件，我们可以手动创建 index.html，通过`HtmlWebpackPlugin`插件配置相应的属性，如：title、template; 我们也可以完全让`HtmlWebpackPlugin`插件自行创建该页面

```js
// 安装
npm install --save-dev html-webpack-plugin

yarn add --dev html-webpack-plugin

// 基本使用
var HtmlWebpackPlugin = require('html-webpack-plugin');

module.exports = {
 ...
  plugins: [new HtmlWebpackPlugin()]
};
```

### DefinePlugin

[插件文档](https://www.webpackjs.com/plugins/define-plugin/)

`DefinePlugin` 允许创建一个在编译时可以配置的全局常量。这可能会对开发模式和发布模式的构建允许不同的行为非常有用。如果在开发构建中，而不在发布构建中执行日志记录，则可以使用全局常量来决定是否记录日志。这就是 `DefinePlugin` 的用处，设置它，就可以忘记开发和发布构建的规则

`DefinePlugin` 为 webpack 自带插件

```js
// 使用
new webpack.DefinePlugin({
  PRODUCTION: JSON.stringify(true),
  VERSION: JSON.stringify("5fa3b9"),
  BROWSER_SUPPORTS_HTML5: true,
  TWO: "1+1",
  "typeof window": JSON.stringify("object"),
});
// 访问
console.log("Running App version " + VERSION);
if (!BROWSER_SUPPORTS_HTML5) require("html5shiv");
```

### CleanWebpackPlugin

[插件文档](https://github.com/johnagan/clean-webpack-plugin)

webpack 每次构建前自动清空构建目录

```js
// 安装
npm install --save-dev clean-webpack-plugin

yarn add --dev clean-webpack-plugin

// 基本使用
const { CleanWebpackPlugin } = require("clean-webpack-plugin");

module.exports = {
 ...
  plugins: [new CleanWebpackPlugin()]
};
```

### HotModuleReplacementPlugin

[插件文档](https://www.webpackjs.com/plugins/hot-module-replacement-plugin/)

模块热替换插件，也被称为`HMR`

```js
new webpack.HotModuleReplacementPlugin({
  // Options...
});
```
