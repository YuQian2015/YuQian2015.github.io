---
title: 【前端开发日常 - 18】快速配置webpack开发环境
date: 2020-04-03 18:22:36
tags:
---

## 检查环境
``` shell
$ node -v
v8.10.0

$ npm -v
5.6.0

$ yarn -v
1.13.0
```
## 创建项目
``` shell
$ mkdir webpack-start

$ cd webpack-start

$ npm init
```
根据提示输入项目信息，比如：
``` shell
Press ^C at any time to quit.
package name: (webpack-start)
version: (1.0.0)
description: 快速开始webpack开发
entry point: (index.js) src/index.js
test command:
git repository:
keywords: webpack
author: moyufed
license: (ISC)
About to write to F:\project\webpack-start\package.json:

{
  "name": "webpack-start",
  "version": "1.0.0",
  "description": "快速开始webpack开发",
  "main": "src/index.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" &amp;&amp; exit 1"
  },
  "keywords": [
    "webpack"
  ],
  "author": "moyufed",
  "license": "ISC"
}


Is this ok? (yes)
```
## 安装 webpack 和 webpack-cli
现在都已经 0202 年，webpack已经是到了4的版本了，不再与webpack-cli放在同一个仓库，因此安装完 webpack 之后还需要安装 webpack-cli 搭配食用才会香。参考文章：<a href="https://stackoverflow.com/questions/49092291/the-cli-moved-into-a-separate-package-webpack-cli">https://stackoverflow.com/questions/49092291/the-cli-moved-into-a-separate-package-webpack-cli</a>
那么来看看安装命令：
``` shell
# 使用npm
$ npm install --save-dev webpack webpack-cli

# 使用yarn
$ yarn add -D  webpack webpack-cli
```
这命令里面的 --save-dev 和 -D 都表示将包安装到开发环境， -D 是 --dev 的简写。
执行安装之后，可以看到package.json文件里面多了两个依赖，是我们的开发环境需要使用的：
``` javascript
// ...
  "devDependencies": {
    "webpack": "^4.42.0",
    "webpack-cli": "^3.3.11"
  }
// ...
```
## 配置webpack
在项目的根目录下创建一个 webpack.config.js 文件，当我们使用 webpack 时会自动查找这个文件里面的配置信息。
创建好配置文件之后，就需要编写一个简单的配置，让webpack可以正常工作：
``` javascript
const path = require('path');

module.exports = {
    entry: './src/index.js',
    output: {
        path: path.resolve(__dirname, 'dist'),
        filename: 'bundle.js'
    }
}
```
上面的配置代码指定了入口文件，也指定了打包输出的文件目录和文件名，接下来就是去建立入口文件&nbsp;src/index.js ，然后在里面写自己的 JavaScript 代码 。
## 启动项目
在package.json中，找到script这一项，增加webpack的执行命令：
``` javascript
// ...
  "scripts": {
    "build": "webpack"
  },
//
```
添加好了之后，我们可以在命令行执行命令：
``` shell
# npm
$ npm run build

# yarn
$ yarn build
```
执行完成之后，就可以看到项目产生了一个 dist 目录，里面生成了 bundle.js 文件，里面包含了我写的 JavaScript 代码。
## 配置开发服务器
要配置开发服务器（devServer），我们需要安装&nbsp;<a href="https://github.com/webpack/webpack-dev-server">webpack-dev-server</a>&nbsp;，具体可以参考文档&nbsp;<a href="https://webpack.docschina.org/configuration/dev-server/">https://webpack.docschina.org/configuration/dev-server/</a>&nbsp;，
首先来安装：
``` shell
# npm
$ npm install --save-dev webpack-dev-server

# yarn
$ yarn add -D webpack-dev-server
```
执行安装之后，需要来配置服务器，可以在webpack的配置文件里面增加 devServer 配置：
``` javascript
const path = require('path');

module.exports = {
    entry: './src/index.js',
    output: {
        path: path.resolve(__dirname, 'dist'),
        filename: 'bundle.js'
    },
    devServer: {
        port: 8080,
        hot: true,
        open: true
    }
}
```
上面配置了开发服务器启动的端口，hot 用来设置开启热模块替换， open 来设置编译之后打开浏览器预览。现在我们进行了简单的devServer配置，还需要增加启动的命令：
``` javascript
// ...
  "scripts": {
    "start": "webpack-dev-server",
    "build": "webpack"
  },
// ...
```
通过执行 start 命令可以启动开发服务器。
## 增加HTML模板
需要使用到 html-webpack-plugin 来使用生成HTML，&nbsp;<a href="https://webpack.docschina.org/plugins/html-webpack-plugin/">https://webpack.docschina.org/plugins/html-webpack-plugin/</a>&nbsp;，执行命令安装：
``` shell
# npm
$ npm install --save-dev html-webpack-plugin

# yarn
$ yarn add -D html-webpack-plugin
```
安装好之后，我们就需要去使用插件了，在webpack.config.js中，引入这个插件，然后在plugin里面使用：
``` javascript
const path = require('path');
const HtmlWebpackPlugin = require('html-webpack-plugin');

module.exports = {
    entry: './src/index.js',
    output: {
        path: path.resolve(__dirname, 'dist'),
        filename: 'bundle.js'
    },
    devServer: {
        port: 8080,
        hot: true,
        open: true
    },
    plugins: [new HtmlWebpackPlugin()]
}
```
使用了插件之后，我们可以再次启动开发服务器，这时候插件就自动会生成HTML了，并且还会自动引入我们的bundle.js，如果我们需要自定义HTML的模板，还可以对插件进行配置，配置的文档：&nbsp;<a href="https://github.com/jantimon/html-webpack-plugin#options">https://github.com/jantimon/html-webpack-plugin#options</a>&nbsp;，下面我们对HTML进行简单配置：
``` javascript
// ...
    plugins: [new HtmlWebpackPlugin({
        title: 'webpack-start',
        template: 'index.html',
    })]
// ...
```
相应的，我们需要在项目根目录新建一个 index.html 文件，使用 html-webpack-plugin 的配置：
``` html
<!DOCTYPE html>
<html>
  <head>
    <meta charset="utf-8"/>
    <title><%= htmlWebpackPlugin.options.title %></title>
  </head>
  <body>
  </body>
</html>
```
上面的代码中使用了 webpack 配置里面的 title ，更多配置可以参考：&nbsp;<a href="https://github.com/jantimon/html-webpack-plugin#configuration">https://github.com/jantimon/html-webpack-plugin#configuration</a>&nbsp;。
## 安装babel
这里使用 babel7，需要注意的是，我们要安装 webpack 的 babel-loader 才能使用 babel，但是 babel-loader 的版本是和 babel 不一样的，比如 babel7 要使用的 babel-loader 是8.x版本，如果安装相同的版本，编译时会报错。
因此在执行安装之后，注意检查 package.json 里面的 babel-loader 是什么版本。
具体参考：&nbsp;<a href="https://www.webpackjs.com/loaders/babel-loader/">https://www.webpackjs.com/loaders/babel-loader/</a>&nbsp;。
安装babel-loader：
``` shell
# npm
$ npm install --save-dev babel-loader

# yarn
$ yarn add -D babel-loader

```
检查安装的 babel-loader 的版本，安装下来是 8.x 的版本，对应的 babel-core 版本是 7 ：
``` javascript
  "devDependencies": {
    "babel-loader": "^8.0.6",
    // ...
  }
```
安装babel：
``` shell
# npm
$ npm install --save-dev @babel/core

# yarn
$ yarn add -D @babel/core
```
安装  @babel/preset-env，这将使得我们的项目支持 JavaScript 的ES6特性：
``` shell
# npm
$ npm install --save-dev @babel/preset-env

# yarn
$ yarn add -D @babel/preset-env
```
## 配置babel
``` javascript
module: {
  rules: [
    {
      test: /\.js$/,
      exclude: /(node_modules|bower_components)/,
      use: {
        loader: 'babel-loader',
        options: {
          presets: ['@babel/preset-env']
        }
      }
    }
  ]
}
```
上面的配置中，我们对 JavaScript 文件使用了 babel ，并且过滤了我们的依赖安装目录，而在&nbsp;presets 中我们使用了@babel/preset-env ， 这会根据我们的编译目标浏览器来使用对应的兼容，从而将ES6代码转换成ES5。
我们来检测一下有没有生效，首先在我们的index.js代码里面打入一个断点：
``` javascript
class Demo {
    constructor() {
        this.init();
    }

    init() {
        debugger
        console.log(123)
    }
}

new Demo();
```
然后我们在浏览器进入断点查看，开始babel配置之前的代码，发现我们的代码没有进行转换，而我们的浏览器能够正常打印log主要是因为该浏览器已经支持ES6的语法：
``` javascript
class Demo {
    constructor() {
        this.init();
    }

    init() {
        debugger
        console.log(123)
    }
}

new Demo();
```
开启babel之后的代码，我们发现已经经过了转化，成为了ES5的代码：
``` javascript
var Demo = /*#__PURE__*/function () {
  function Demo() {
    _classCallCheck(this, Demo);

    this.init();
  }

  _createClass(Demo, [{
    key: "init",
    value: function init() {
      debugger;
      console.log(123);
    }
  }]);

  return Demo;
}();

new Demo();
```
关于 @babel/preset-env&nbsp;可以参考：&nbsp;<a href="https://www.babeljs.cn/docs/babel-preset-env">https://www.babeljs.cn/docs/babel-preset-env</a>&nbsp;。
## 使用类属性
我们接着来修改 index.js 代码，这次我们来使用类的属性，如下：
``` javascript
class Demo {

    text = '123';

    constructor() {
        this.init();
    }

    init() {
        console.log(this.text);
    }
}

new Demo();
```
我们发现，这是 webpack 编译报错了：
``` shell
ERROR in ./src/index.js
Module build failed (from ./node_modules/babel-loader/lib/index.js):
SyntaxError: G:\webpack-start\src\index.js: Support for the experimental syntax 'classProperties' isn't currently enabled (3:10):
```
因此我们需要解决现在的配置不能使用 classProperties 的问题。为了解决这个问题我们需要安装一个插件，在babel7里面，我们使用的是 @babel/plugin-proposal-class-properties 。
注意：在 babel7 里面我们的插件基本上都是在 @babel 这个空间下面。
先来安装插件:
``` shell
# npm
$ npm install --save-dev @babel/plugin-proposal-class-properties

# yarn
$ yarn add -D @babel/plugin-proposal-class-properties
```
然后在我们的配置里面使用插件：
``` javascript
// ...
module: {
        rules: [
            {
                test: /\.js$/,
                exclude: /(node_modules|bower_components)/,
                use: {
                    loader: 'babel-loader',
                    options: {
                        presets: [
                            '@babel/preset-env'
                        ],
                        plugins: [
                            '@babel/plugin-proposal-class-properties'
                        ]
                    }
                }
            }
        ]
    },
// ...   
```
接着我们来启动编译，代码正常运行。
关于@babel/plugin-proposal-class-properties具体可以参考文档：&nbsp;<a href="https://babeljs.io/docs/en/babel-plugin-proposal-class-properties">https://babeljs.io/docs/en/babel-plugin-proposal-class-properties</a>
## 使用async/await
我们接着来修改index.js里面的代码，这次要在代码里面async/await：
``` javascript
class Demo {

    text = '123';

    constructor() {
        this.init();
    }

    async init() {
        try {
            await this.wait();
        } catch(error) {
            console.log(error);
        }
        console.log(this.text);
    }

    wait() {
        return new Promise((resolve, reject) =&gt; {
            setTimeout(() =&gt; {
                if(Math.random() &gt; 0.5) {
                    resolve()
                } else {
                    reject(new Error('&lt;=0.5'))
                }
            }, 1000);
        })
    }
}

new Demo();
```
保存，编译之后，我们发现浏览器报出错误：
``` javascript
Uncaught ReferenceError: regeneratorRuntime is not defined
```
为了解决这个问题，我们需要安装&nbsp;@babel/plugin-transform-runtime：
``` shell
# npm
$ npm install --save-dev @babel/plugin-transform-runtime

# yarn
$ yarn add -D @babel/plugin-transform-runtime
```
继续，我们去修改配置，添加这个babel插件：
``` javascript
 // ...   
		module: {
        rules: [
            {
                test: /\.js$/,
                exclude: /(node_modules|bower_components)/,
                use: {
                    loader: 'babel-loader',
                    options: {
                        presets: [
                            '@babel/preset-env'
                        ],
                        plugins: [
                            '@babel/plugin-proposal-class-properties',
                            '@babel/plugin-transform-runtime'
                        ]
                    }
                }
            }
        ]
    },
// ...
```
重新编译，我们发现代码已经能够正常运行。
关于 @babel/plugin-transform-runtime 具体可以参考文档：<a href="https://babeljs.io/docs/en/babel-plugin-transform-runtime">https://babeljs.io/docs/en/babel-plugin-transform-runtime</a>&nbsp;。
## 结尾
本次只介绍从安装 webpack 到配置 babel ，从而满足我们的开发所需，关于 webpack 和 babel 的配置还有很多内容，这将会在以后继续补充进来。
