---
title: JavaScript的this关键字
date: 2020-03-30 22:15:14
tags:
---
## JavaScript的this关键字

`this` 关键字指向一个对象，该对象是JavaScript当前代码执行的小块。换而言之，每个JavaScript方法执行的时候都有一个执行上下文（execution context），就是我们的 this。执行上下文体现了当前方法是如何被调用的。

为了理解 `this` 关键字，我们首先要关注方法是何时、何处、以何种方式调用的，而不是关注方法在何处以何种方式声明。

```javascript
function bike() {
  console.log(this.name);
}

var name = "Ninja";
var obj1 = { name: "Pulsar", bike: bike };
var obj2 = { name: "Gixxer", bike: bike };

bike();           // "Ninja"
obj1.bike();      // "Pulsar"
obj2.bike();      // "Gixxer"
```

在上面的代码中， `bike()` 方法的作用就是打印 `this.name` ，也就是说打印出当前执行上下文的 `name` 属性的值。当 `bike()`  方法被调用时，打印出 “Ninja” ，因为在没有指定的情况下，执行上下文都会指向**全局对象**，而在全局对象上面，我们定义了一个 `name` 的属性，值是 “Ninja”。

同样的，在 `obj1().bike()` 中打印了 “Pulsar” ，因为 `bike()` 方法是以 `obj1` 作为执行上下文来调用的， `this.name`  实际上就是 `obj1.name` 。同理 `obj2.bike()` 方法的执行上下文就是 `obj2` 。

## This的默认绑定和隐式绑定

**默认绑定：**如果我们的代码是在严格模式（strict mode）下， `this` 的值就是 `undefined` ，否则， `this` 指代了**全局对象** 。

当单独使用时，它指的是**全局对象** ，浏览器中的**全局对象**其实就是 window 对象 [**object Window**] ，示例：

```javascript
var a = this; 
```

当在一个方法中使用，this指向全局对象，示例：

```javascript
function func() {
    return this;
}
```

注意：严格模式不允许默认绑定，所以 `this` 将会是 `undefined` ：

```javascript
"use strict";
function func() {
    return this;
}
```

**隐式绑定：**当使用一个对象的属性调用一个方法时，该对象就成了方法的执行上下文 `this`。

```javascript
var obj1 = {
  name: "Pulsar",
  bike: function() {
    console.log(this.name);
  }
}
var obj2 = { name: "Gixxer", bike: obj1.bike };
var name = "Ninja";
var bike = obj1.bike;

bike();           // "Ninja"
obj1.bike();      // "Pulsar"
obj2.bike();      // "Gixxer"
```

上面的代码中，直接调用 `bike()` 就是默认绑定， `obj1.bike()` 和 `obj2.bike()` 就是隐式绑定。

`bike` 方法是 `obj1` 的一个属性，但是当我们执行 `obj2.bike()` 时，执行上下文是 `obj2  ` ，因此`obj2.name` 的值被打印出来。

所以，清楚什么时间、什么地方、以何种方式调用方法是很重要的，而方法的声明并不影响。

## This的显式绑定和硬绑定

**显式绑定：**如果我们使用 `call` 和 `apply` 来调用方法，那么方法将以第一个参数来作为它的执行上下文。

```javascript
function bike() {
  console.log(this.name);
}

var name = "Ninja";
var obj = { name: "Pulsar" }

bike();           // "Ninja"
bike.call(obj);   // "Pulsar"
```

上面的代码中，我们传递了一个对象 `obj` 作为 `call()` 的参数来调用 `bike` 方法，`obj` 就被指派到了 `this` ，输出`obj.name` 的值 “Pulsar” 。

**硬绑定：**无论我们怎么调用方法，指定 `this` 对象始终不变。

```javascript
var bike = function() {
  console.log(this.name);
}
var name = "Ninja";
var obj1 = { name: "Pulsar" };
var obj2 = { name: "Gixxer" };

var originalBikeFun = bike;
bike = function() {
  originalBikeFun.call(obj1);
};

bike();           // "Pulsar"
bike.call(obj2);  // "Pulsar"
```

上面的代码中使用了`originalBikeFun.call(obj1); ` ，使得 `bike()` 和 `bike.call(obj2)` 都输出`obj1.name`  的值。

## JavaScript中的new关键字

任何方法之前的 `new` 关键字会调用方法的 `constructor` ，然后产生下面的结果：

- 产生一个新的空的对象；
- 该新对象关联到方法的 `prototype` 属性；
- 该新对象被绑定为 `this` 关键字，将作为方法的执行上下文；
- 如果方法没有返回任何值，那将隐性的返回 this 对象。

```javascript
function bike() {
  var name = "Ninja";
  this.maker = "Kawasaki";
  console.log(this.name + " " + maker);  // undefined Bajaj
}

var name = "Pulsar";
var maker = "Bajaj";

obj = new bike();
console.log(obj.maker);                  // "Kawasaki"
```

上面的代码中，`bike` 方法前面使用了 `new` ，因此它将创建一个新对象连接到 `bike` 方法的原型链, 然后该对象绑定到 `this` 从而被返回。因此返回的 `this` 对象被分配到 `obj` 然后`console.log(obj.maker)` 打印出 “Kawasaki” 。

同样是上面的代码，方法里面的 `this.name` 没有打印出 “Ninja" 或者 “Pulsar” ，反而是 `undefined` 。因为 `name` 是在方法里面被声明，和 `this.name ` 是完全不同的。同样， `this.maker` 和 `maker` 也是不同的。

## This绑定顺序

- 首先检查方法是否用 `new` 来调用;
- 然后检查方法是否是被 `call()` 或者 `apply()` 方法来调用;
- 其次检查方法是否是通过对象上下文来调用;
- 默认全局对象（严格模式是 `undefined` ）。

原文链接：https://codeburst.io/all-about-this-and-new-keywords-in-javascript-38039f71780c

参考链接：https://segmentfault.com/a/1190000004460913