---
title: 什么是DOM的事件委托
date: 2019-10-29 15:23:54
tags: [前端]
toc: true
---

### DOM的事件委托

DOM的事件委托（Event delegation）是通过事件 ”冒泡“ (event propagation) 来用单个父节点而非多个子节点响应 [UI Events](https://www.w3.org/TR/uievents/) 的技巧。

<!--more-->

#### 冒泡

什么是冒泡？事件被绑定到它的目标节点 `EventTarget` 上，当事件监听器发现事件被触发后，将顺着目标节点的父级链逐层检查，并触发额外的事件监听器，这种动作持续向父级传递，包括 `Document` 。

有了事件冒泡，事件委托就有了基础，可以将事件的处理程序绑定给一个父级元素，当任何子节点触发了匹配的父级元素绑定的事件，即可触发父级节点的处理程序，这就是 **事件委托** 。

栗子：

```html
<ul onclick="alert(event.type)">
    <li>One</li>
    <li>Two</li>
    <li>Three</li>
</ul>
```

上面示例中，即时没有给每个 `<li>` 绑定事件，也会在点击任何一个 `<li>` 节点后弹出 "click" 。

#### 优点

1. 减少事件绑定，事件委托将很多子元素的事件绑定都集中到一个通用的父元素，使得动态创建和移除的元素更加方便，不需要考虑元素的事件绑定逻辑。假设需要对 `<li>` 标签进行增加，只管进行操作就行，不用增加元素的 “onclick” 事件。
2. 减少事件监听使用的内存，这可能对那些经常重新加载的小页面效果不明显，但是对需要长时间运行的应用很重要。因为当元素被从 DOM 中移除之后很难追踪它对内存的使用，造成内存泄露，这是由元素的事件绑定造成的。有了事件委托，在移除子元素之后不用担心没有解除绑定事件。

#### 使用

##### JavaScript

假设有一个父级元素 `ul` 和若干子元素 `li` ：

```html
<ul id="parent">
	<li id="child-1">Child 1</li>
	<li id="child-2">Child 2</li>
	<li id="child-3">Child 3</li>
	<li id="child-4">Child 4</li>
	<li id="child-5">Child 5</li>
	<li id="child-6">Child 6</li>
</ul>
```

现在希望每一个 `li` 被点击时可以获取到它的内容，当事件冒泡到 `ul` 时，检查事件的 `target` 属性来获得被点击的节点，从而获取内容：

```js
// 获取元素并且添加监听事件
document.getElementById("parent").addEventListener("click", function(e) {
    // e.target 是被点击的元素
    if(e.target && e.target.nodeName == "LI") {
        // List item found!  Output the ID!
        alert(e.target.innerHTML+ " was clicked!");
    }
});
```

##### jQuery.delegate

```html
<ul id="jQueryDelegate">
    <li>Item 1</li>
    <li>Item 2</li>
    <li>Item 3</li>
    <li>Item 4</li>
</ul>
```

```js
$("#jQueryDelegate").delegate("li", "click", function () {
    $("#jQueryDelegate").append("<li>jQuery.delegate new item</li>");
});
```

##### jQuery.on

```html
<ul id="jQueryOn">
    <li>Item 1</li>
    <li>Item 2</li>
    <li>Item 3</li>
    <li>Item 4</li>
</ul>
```

```js

$("#jQueryOn").on("click", "li", function () {
	$("#jQueryOn").append("<li>jQuery.on new item</li>");
});
```



#### 参考资料

[What is DOM Event delegation?](https://stackoverflow.com/questions/1687296/what-is-dom-event-delegation) *By* [Crescent Fresh](https://stackoverflow.com/users/45433/crescent-fresh)

[How JavaScript Event Delegation Works](https://davidwalsh.name/event-delegate) *By* [David Walsh](http://davidwalsh.name/) 