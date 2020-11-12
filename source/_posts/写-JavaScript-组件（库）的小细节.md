---
title: 写 JavaScript 组件（库）的小细节
date: 2020-11-12 18:14:30
tags: [前端]
---


开发一个库要考虑那些？先来快速看一下：

- 设置 GitHub
- 配置 npm 并且创建 package.json
- 创建组件库并添加依赖
- 推代码到 GitHub
- 发布 NPM 包
- 发布一个版本到 GitHub
- 发布一个新版本到 NPM
- 发布一个 beta 版本
- 设置好单元测试（如Mocha 和 Chai）
- 进行单元测试
- 自动管理发布版本（semantic-release）
- 提交规范（如commitizen）
- 提交新特性（如commitizen）
- 持续集成（如TravisCI）
- 提交前自动测试（如ghooks）
- 代码覆盖率统计（如istanbul）
- 代码覆盖率检查
- 代码覆盖率报告分析
- README（如添加徽章等）
- 添加 ES6 支持
- 测试支持 ES6（Mocha 和 Babel）
- 限制编译分支
- 浏览器版本编译

下文的内容不会对上面的点做介绍，只在从一些容易忽略的小点进行介绍。

### 任务明确

明确写的库或组件目的是什么，一个库应该专注于处理好特定的任务。比如 React 和 Vue 都是围绕着DOM操作来进行抽象的，又比如 Angular 专注于DOM抽象、HTTP网络、依赖注入等并且提供一个完整的应用框架。

先来看下面这个库，用来判断是否存在用户 home 目录，引入了 2 个库，并且只有 1 行有效代码，这适得其反：

```javascript
'use strict';
var osHomedir = require('os-homedir');
var osTmpdir = require('os-tmpdir');
module.exports = osHomedir() || osTmpdir();
```

再来看 [egg-jwt](https://github.com/okoala/egg-jwt) 的这个中间件，只是单纯的使用了 `koa-jwt2` 并且设置了参数而已，这也是为什么我直接使用 `koa-jwt2` 的原因：

```javascript
'use strict';

const koajwt = require('koa-jwt2');

module.exports = options => {
  return koajwt(options);
};
```

总的来说就是一个库要解决实际问题，而不是没有意义或是造成繁琐。

### 向下兼容

编写组件时，应尽量考虑向下兼容，不然会造成使用者的很多疑惑，npm 在安装时会自动决定版本：

- ~ 会匹配最近的小版本（修订版本）依赖包，比如 `~1.2.3` 会匹配所有 `1.2.x` 版本，但是不包括 `1.3.0` 
- ^ 会匹配最新的大版本（次版本）依赖包，比如 `^1.2.3` 会匹配所有 `1.x.x` 的包，包括 `1.3.0` ，但是不包括 `2.0.0`

虽然 npm 现在引入了 [package-lock.json](https://docs.npmjs.com/files/package-locks) 这样，但是任然没有办法解决安装新包导致的版本问题。

即使你遵循了 [semver](https://semver.org/)（语义化版本规范）发布了对应的主版本号，也要尽量避免 *breaking changes* （破坏性升级），如果你正在使用一个库并发现了 bug，并且注意到 bug 在以后的主版本中得到了修复，除非库作者维护了多个主要版本，否则你可能需要重构你的代码来使用最新的主版本。遵循 semver 列出的设计，向下兼容应该更容易。

关于语义化版本规范版本号格式，下面引用文章：[semver 语义化版本规范](https://www.jianshu.com/p/a7490344044f) 中的一段介绍：

> 主版本号[MAJOR].次版本号[MINOR].修订号[PATCH]，版本号递增规则如下：
>
> 1. 主版本号：当做了不兼容的 API 修改，
> 2. 次版本号：当做了向下兼容的功能性新增，
> 3. 修订号：当做了向下兼容的问题修正。
>
> 先行版本号及版本编译信息可以加到“主版本号.次版本号.修订号”的后面，作为延伸。
>
> 当主版本号升级后，次版本号和修订号需要重置为0，次版本号进行升级后，修订版本需要重置为0。

### 轻量和灵活

尝试使用较少的代码解决问题，可以减少向下兼容成本和 API 的提供。轻量灵活的组件让使用者少面临一些问题，并且节省带宽。

举个栗子：

假设你在开发一个下拉组件要支持 auto-complete，与其考虑如何让组件发送请求、展示数据，不如考虑如何去支持 auto-complete。下面的代码是一个例子：

```javascript
let dropdown = new Dropdown();
input.addEventListener('input', (e) => {
   let value = e.target.value;
   fetch(`/search?value=${value}`).then(res => {
        dropdown.setItems(res);
        dropdown.showList();
   });
});
```

使用者只需要监听 `input` 的内容，发送请求到他们的服务器获得结果，然后调用组件的 `setItems` 和 `showList`

来展示。如果只需要本地自动补全，可以将请求本身换成一个本地的数据查找即可。

### 避免代码隐患

多考虑组件的输入输出会存在的隐患，考虑使用者的使用场景。

举个栗子：

假设你在创建一个组件，使用者在使用时需要传递一个对象数组，数组每项包括  `title` 和 `value` ，当他们点击每一项时，将传递对应项的  `title` 和 `value` 属性到 `callback` 方法里面。

那么问题来了：

假如使用者传递了你不支持的 key 将如何？这个 key 会传递到给定的 callback 吗？假设未来你将使用到这个 key 呢？你在组件里是使用同一个对象还是克隆的对象？假设使用者传递了一个数组并且在别处修改了这个数组会怎么样？组件需要因为数组的变化而更改显示吗？

这都是非常容易忽略的问题，但是却很重要。这些异常疏漏很容易产生 bug，如果你修复了这些 bug（比如通过克隆使用传参），这将导致他人的应用面临崩溃的风险，因此在组件发布之前需要尽力排查出这些问题。

### 思考组件的细节

#### 输入输出

一个组件通常都需要设计合理的 `input` 和 `output` ，在编写组件时，需要考虑恰当的输入值和类型，是否需要回调，并且对这些传入的值进行类型的校验/限制，因为你无法强制使用者传或是不传，也不能确定强制用户怎么传递。

#### 默认的参数

尽可能的为组件本身的属性、数值设置默认值，当用户没有传递任何值时，确保组件的正常工作。

#### React 中的 dumb组件 和 smart组件

在react中，只会接收props，根据props进行渲染的组件称为Dumb组件。Dumb组件不依赖除了 React.js 和Dumb组件以外的内容，Dumb组件是最纯粹的，可复用性最好的组件。单靠Dumb组件无法构建应该程序，因为它们除props外，不依赖其他外部的内容，因而无法获取数据。这就需要另一种组件，它们非常“聪明”，专门从事数据相关的应用逻辑，如发送XMLHttpRequest，这种组件称为Smart组件。

#### 贴合原生DOM的属性设置

使用者会遵循习惯和常识来使用你的组件，在设计输入属性和回调函数时，应尽量语义化、标准化。并且贴合原生 DOM，比如设置一个 Input 组件，我们考虑的属性应该是 `value` 、 `onchange` 、 `placeholder` 等，这些都是使用者非常熟悉的，可以本能的来使用组件。

#### 提供可定制的方法

很多使用者在使用组件时，都需要可以定制样式或特定的结构，这需要组件开发阶段考虑进来。

### 简化 HTML 和 CSS

首先，为你的 HTML 样式增加合适的命名空间，这将确保你的组件样式不会影响到其它组件，也给了其他使用者一个清晰的使用和定制方向。

BEM 是一种前端命名方法论，主要是针对 CSS，意思是块（Block）、元素（Element）、修饰符（Modifier）的简写。这种命名方法让 CSS 便于统一团队开发规范和方便维护。*（例子： MyComponent-element_modifier_value）*。

举个栗子：

#### HTML

```html
<button class="button">
Normal button
</button>
<button class="button button--state-success">
Success button
</button>
<button class="button button--state-danger">
Danger button
</button>
```

#### CSS

```css
.button {
    display: inline-block;
    border-radius: 3px;
    padding: 7px 12px;
    border: 1px solid #D5D5D5;
    background-image: linear-gradient(#EEE, #DDD);
    font: 700 13px/18px Helvetica, arial;
}
.button--state-success {
    color: #FFF;
    background: #569E3D linear-gradient(#79D858, #569E3D) repeat-x;
    border-color: #4A993E;
}
.button--state-danger {
    color: #900;
}
```

#### 优点

模块化：

- 块样式永远不依赖于页面上的其他元素，因此你不会遇到级联问题。
- 你还可以将已完成项目中的块转移到新项目。

可重用性：

- 以不同方式组合独立块，并智能地重用它们，减少了你必须维护的 CSS 代码量。
- 有了一套样式指南，你可以构建一个块库，使你的 CSS 超级有效。

很多组件库往往提供了复杂的组件效果，个人倾向于用最少的样式，最少的层级来让你的组件工作，因为要定制/覆盖组件样式对于使用者来说是非常麻烦的事情，往往导致很多问题。

考虑为使用者提供一些 API 来注入他们的 DOM，而不是只提供特定的配置项。

回到前面提到的下拉组件，比起提供图标，更好的方式是提供方法给用户返回 DOM 插入到每一项，这样只需要提供一个方法，就能实现灵活定制列表图标

### 跟进浏览器标准

Web 开发技术日新月异，新的技术总是带来更多的便利，在使用新的特性时，需要考虑兼容性。新的标准被提出，旧的标准也可能移除，现在的代码可能在未来收到影响，因此需要跟进浏览器标准，以确保自己的组件库能够运行在用户的平台。

### 减少不必要的依赖

- 是否必须使用lodash？
- 是否能使用原生DOM API来代替 jQuery？
- 是否能够使组件支持多个前端库，而不仅仅是 React？

当你引入依赖，意味使用者需要自己清楚这些依赖的稳定性、大小、许可权等等问题，是否对自己的现有代码造成影响。减少依赖可以让自己的组件更通用，适配性更强，更适合于在不同的前端技术栈里面封装/使用。

### 致力于最小

当在写一个组件时，需要将库的体积尽可能的减小。浏览器执行 JavaScript 时需要解析编译，100KB的 JavaScript 和 100KB 的图片是完全不一样的，不管是否使用 gzip 压缩或者 HTTP2，JavaScript 执行时都需要解析编译成机器语言，这些都需要耗费时间。

如果是使用 ES6 和 Babel，即使写的代码变少了，最终生成的代码也是巨量的，这点也要考虑进去。

### 导出版本

#### ES6 

模块的引入不光有传统的 AMD（异步模块定义）规范 和 CommonJS 规范，还有现在的 ES6 模块定义，可以考虑导出支持 ES6 import 的库，让使用者决定以哪种方式来引入，并且支持 [tree-shaking](https://rollupjs.org/) 编译。

#### UMD

UMD（Unified Module Definition 通用模块定义） 让我们可以在浏览器使用 `<script>`  引入代码也可以使用 `require` 引入，但是UMD打包出来的文件会添加很多可能不会用到的代码，因此导出版本时也需要考虑到这一点。一些注意事项可以参考：[How to write and build JS libraries in 2018](https://medium.com/@kelin2025/so-you-wanna-use-es6-modules-714f48b3a953) 。

关于模块定义，可以参考 [认识AMD、CMD、UMD、CommonJS](https://www.cnblogs.com/humin/p/5389901.html) 一文。

### 测试用例

在组件开发中，测试用例是必要的，这可以让其他使用者放心使用你的组件库而不需自己测试，也为自己提供机制确保不会破坏向下兼容。

在写测试用例的时候，不要只做简单的检查，需要详细的检查确保功能按照预期工作，产生的异常、适配等问题需要反复测试。

当你修复了一个 bug，也需要写一个测试用例来验证 bug 不会复现，目的就是减少异常的产生。

### 学习使用许可

MIT 是 JavaScript 开源社区标配的一种许可标准，如果使用了像 GPL（General Public License 通用公共许可证） 这种模棱两可的许可，可能会造成一定的歧义。这将会影响你的组件库的使用量，应尽量为自己和使用者减少麻烦。 

关于开源协议，详情查看：[五种开源协议的比较(BSD，Apache，GPL，LGPL，MIT)](http://www.ha97.com/833.html) ，这里摘抄一段：

> **MIT(MIT)**
>
> MIT是和 BSD一样宽范的许可协议，作者只想保留版权，而无任何其他的限制，也就是说，你必
> 须在你的发行版里包含原许可协议的声明，无论你是以二进制发布的还是以源代码发布的。

### 总结

上面是一些关于编写JavaScript库或组件的思考，要做的东西很多，但目的都是为了写更少的代码，让更多人受益。

## 参考资料

- [Tips on Writing Good JavaScript Libraries](https://medium.com/@PepsRyuu/tips-on-writing-good-javascript-libraries-e3c3068ec705)