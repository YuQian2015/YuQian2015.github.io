---
title: 【前端开发日常 - 3】让create-react-app支持@装饰器语法
date: 2020-09-16 16:52:39
tags: [React, 前端]
---

### 需求背景
目前在做的一个 react 项目是使用 `create-react-app` 创建的，我需要在里面使用 `mobx` 和 `mobx-react` 管理状态，`mobx-react` 支持使用装饰器的方式来书写，而 `create-react-app` 目前还没有内置的装饰器支持，所以在启动项目时会报错。

``` shell
Support for the experimental syntax 'decorators-legacy' isn't currently enabled
```
### 解决方案

使用 [`react-app-rewired`](https://github.com/timarney/react-app-rewired) 修改 `create-react-app` 的配置，使其支持装饰器语法。

> 此工具可以在不 'eject'  也不创建额外 react-scripts 的情况下修改 create-react-app 内置的 webpack 配置，然后你将拥有 create-react-app 的一切特性，且可以根据你的需要去配置 webpack 的 plugins, loaders 等。

<!--more-->

### 检查版本

检查 babel 版本，我们可以打开 `node_modules/react-scripts/package.json` 查看安装的 babel 版本，针对不同的版本安装不同插件：

- babel 6 安装 `babel-plugin-transform-decorators-legacy`

- babel 7 安装 `@babel/plugin-proposal-decorators`

可以在 [babel-plugin-transform-decorators-legacy](https://github.com/loganfsmyth/babel-plugin-transform-decorators-legacy) 查看版本区别。

### 安装插件

安装 `react-app-rewired` 以及 babel 插件，本人使用babel 7，所以安装 `@babel/plugin-proposal-decorators` ：

``` shell
$ yarn add --dev react-app-rewired
$ yarn add --dev @babel/plugin-proposal-decorators
```
修改 `package.json` 文件：

``` javascript
"scripts": {
    "start": "react-app-rewired start",
    "build": "react-app-rewired build",
    "test": "react-app-rewired test",
    "eject": "react-scripts eject"
},
```
注意：由于 `react-app-rewire` v2.0  以上版本已经移除了 `getBabelLoader` ，需要使用 [`customize-cra`](https://github.com/arackaf/customize-cra) 库的 `addBabelPlugins` 方法代替。

```shell
The "getBabelLoader" helper has been deprecated as of v2.0. You can use customize-cra plugins in replacement - https://github.com/arackaf/customize-cra#available-plugins
```

### 安装 customize-cra

```shell
$ yarn add --dev customize-cra
```

在项目根目录新增 `config-overrides.js` 文件：

``` javascript
const { override, addDecoratorsLegacy, disableEsLint } = require("customize-cra");

module.exports = override(
    addDecoratorsLegacy(),
    disableEsLint()
)
```
### 启动项目

``` shell
$ yarn start
```

### 参考来源
<a href="https://www.jianshu.com/p/23d9ce9ac718">https://www.jianshu.com/p/23d9ce9ac718</a>
<a href="https://www.cnblogs.com/so-letitgo/p/10010428.html">https://www.cnblogs.com/so-letitgo/p/10010428.html</a>
<a href="https://www.jianshu.com/p/b16f7598c859">https://www.jianshu.com/p/b16f7598c859</a>
<a href="https://www.jianshu.com/p/b841aee4745f">https://www.jianshu.com/p/b841aee4745f</a>
