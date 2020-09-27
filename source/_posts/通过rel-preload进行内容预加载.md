---
title: 通过rel=preload进行内容预加载
date: 2020-09-13 23:14:07
tags: [前端]
toc: true
---

更多详情可以参考，参考 [MDN文档](https://developer.mozilla.org/zh-CN/docs/Web/HTML/Preloading_content) 。

### 需求背景

最近遇到的问题：

- 一个页面加载主要逻辑 `main.js` ，该文件很大，设计很多业务代码，并且依赖部分 JavaScript 插件，在 `main.js` 执行之前需要确保 JavaScript 插件加载完成。
- 需要确保 `main.js` 以较快的速度加载，不能因为 `js` 文件的加载顺序影响首页渲染时间。

<!--more-->

### 解决方案

- 预加载 `main.js` 文件，等到 JavaScript 插件加载完成之后生效。

### 目的

- 在浏览器的主渲染机制介入前预先加载资源。
- 不阻塞页面的初步渲染，进而提升性能。

### 原理

 [`<link>`](https://developer.mozilla.org/zh-CN/docs/Web/HTML/Element/link) 元素的  `rel` 属性的属性值 `preload` 能够让你在HTML页面中 [`<head>`](https://developer.mozilla.org/zh-CN/docs/Web/HTML/Element/head) 元素内部书写一些声明式的资源获取请求，可以指明哪些资源是在页面加载完成后即刻需要的。

### 一般使用

`<link>` 标签最常见的应用情形就是被用来加载CSS文件，进而装饰页面：

```html
<link rel="stylesheet" href="styles/main.css">
```

### 预加载使用

使用`preload`作为`rel`属性的属性值，你还需要通过 `href` 和 `as` 属性指定需要被预加载资源的资源路径及其类型。简单示例：

```html
<head>
  <meta charset="utf-8">
  <title>JS and CSS preload example</title>

  <link rel="preload" href="style.css" as="style">
  <link rel="preload" href="main.js" as="script">

  <link rel="stylesheet" href="style.css">
</head>

<body>
  <h1>bouncing balls</h1>
  <canvas></canvas>

  <script src="main.js"></script>
</body>
```

在这里，我们预加载了CSS和JavaScript文件，所以在随后的页面渲染中，一旦需要使用它们，它们就会立即可用。

`preload` 还有许多其他好处。使用 `as` 来指定将要预加载的内容的类型，将使得浏览器能够：

- 更精确地优化资源加载优先级。

- 匹配未来的加载需求，在适当的情况下，重复利用同一资源。

- 为资源应用正确的[内容安全策略](https://developer.mozilla.org/en-US/docs/Web/HTTP/CSP)。

- 为资源设置正确的 [`Accept`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Accept) 请求头。

### 可以被预加载的资源

许多类型的内容都可以被预加载，一些主要可用的 `as` 属性值列举如下：

- `audio`：音频文件。
- `document`： 一个将要被嵌入到 [`<frame>`](https://developer.mozilla.org/zh-CN/docs/Web/HTML/Element/frame) 或 [`<iframe>`](https://developer.mozilla.org/zh-CN/docs/Web/HTML/Element/iframe) 内部的HTML文档。
- `embed`：一个将要被嵌入到 [`<embed>`](https://developer.mozilla.org/zh-CN/docs/Web/HTML/Element/embed) 元素内部的资源。
- `fetch`：那些将要通过 fetch 和 XHR 请求来获取的资源，比如一个 ArrayBuffer 或 JSON 文件。
- `font`： 字体文件。
- `image`:：图片文件。
- `object`：一个将会被嵌入到 [`<embed>`](https://developer.mozilla.org/zh-CN/docs/Web/HTML/Element/embed) 元素内的文件。
- `script`： JavaScript文件。
- `style`：样式表。
- `track`：WebVTT文件。
- `worker`：一个 JavaScript 的 web worker 或 shared worker 。
- `video`：视频文件。

### 包含一个MIME类型

`<link>` 元素可以接受一个 `type` 属性，在浏览器进行预加载的时候，浏览器将使用 `type` 属性来判断它是否支持这一资源，如果浏览器支持这一类型资源的预加载，下载将会开始，否则便对其加以忽略。

```html
<head>
  <meta charset="utf-8">
  <title>Video preload example</title>

  <link rel="preload" href="sintel-short.mp4" as="video" type="video/mp4">
</head>
<body>
  <video controls>
    <source src="sintel-short.mp4" type="video/mp4">
    <source src="sintel-short.webm" type="video/webm">
    <p>Your browser doesn't support HTML5 video. Here is a <a href="sintel-short.mp4">link to the video</a> instead.</p>
  </video>
</body>
```

在这个实例中，支持MP4格式的浏览器将仅预加载并使用MP4资源，以使得视频播放器的表现尽可能的流畅，或者说，为用户提供更好的响应。而不支持MP4格式的浏览器仍然能够加载视频的WebM版本，但无法体验到预加载带来的良好体验。这个例子展示了预加载机制如何与渐进式增强的哲学进行有机的结合，可以参考 [MDN文档](https://developer.mozilla.org/zh-CN/docs/Web/HTML/Preloading_content) 的详细介绍。

### 脚本预加载

预加载但不执行：

```javascript
var preloadLink = document.createElement("link");
preloadLink.href = "myscript.js";
preloadLink.rel = "preload";
preloadLink.as = "script";
document.head.appendChild(preloadLink);
```

需要执行时：

```javascript
var preloadedScript = document.createElement("script");
preloadedScript.src = "myscript.js";
document.body.appendChild(preloadedScript);
```

### 参考链接

https://developer.mozilla.org/zh-CN/docs/Web/HTML/Preloading_content