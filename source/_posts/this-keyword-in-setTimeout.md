---
title: setTimeout 中的 this 关键字
date: 2017-04-25 22:26:22
tags: [JavaScript]
---

## 概述

本文主要讲述了几种场景下 `setTimeout` 的回调函数的 `this` 绑定出错的几种解决方法。

<!-- more -->

## 举个栗子

```javascript
class Person {
  sayName() {
    console.log("Hello, I'm " + this.name + ".")
  }
  
  constructor(name = "No One") {
    this.name = name
    setTimeout(this.sayName, 1000)
  }
}

var imo = new Person("Imo")

setTimeout(imo.sayName, 2000)
```

> P.S. 上述代码使用了 ES6 语法，这里不做语法上的过多解释。不知所谓的童鞋可以看一下[《30 分钟掌握 ES6 核心内容（上）》](https://segmentfault.com/a/1190000004365693)、[《30 分钟掌握 ES6 核心内容（下）》](https://segmentfault.com/a/1190000004368132)这两篇。 

上述代码定义了一个 `Person` 类，这个类有一个构造函数和一个 `sayName` 方法。其中，`sayName` 方法内部使用了 `this` 关键字，用于引用当前实例的（公有成员）变量 `name`。

上述代码中，有两个 `setTimeout` 函数。第一个 `setTimeout` 存在于 `Person` 类的构造函数当中，其第一个参数是 `this.sayName`；第二个存在于全局作用域中，其第一个参数是 `imo.sayName`。两者都期望在控制台打印 `Hello, I'm Imo.`，但是目前两者都没有达到期望——目前打印的均是 `Hello, I'm .`。出现这种现象的原因是 `sayName` 方法内部的 `this` 没有如期地指向 `imo` 对象，而是错误地指向了全局对象，因而 `this.name` 的值为 `undefined`，对应到字符串是 `''` （空字符串）。

那么，我们如何修改代码使得 `this` 关键字如期地指向 `imo` 对象呢？

## 第二个 `setTimeout`

先解决比较简单的第二个 `setTimeout` 。

有两种方法：

1. `.bind()`
2. 匿名函数包裹

代码如下：

```javascript
// 第二个 setTimeout 用“.bind()”方法修改如下：
setTimeout(imo.sayName.bind(imo), 2000)

// 第二个 setTimeout 用“匿名函数包裹”方法修改如下：
setTimeout(function(){
    imo.sayName()
}, 2000)
```

### “`.bind()`”方法

> Apply 调用模式 - The Apply Invocation Pattern

[Function.prototype.bind() @MDN](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Function/bind)：“`bind()` 方法会创建一个新函数。当这个新函数被调用时，`bind()` 的第一个参数将作为它运行时的 `this` ...”

### “匿名函数包裹”方法

> 方法调用模式 - The Method Invocation Pattern

在外层匿名函数的内部，`sayName` 作为 `imo` 对象的方法被调用，方法内部的 `this` 也就被绑定到 `imo` 对象了。

## 第一个 `setTimeout`

我们尝试上面提到的两种解决方法。

### “`.bind()`”方法

`.bind()` 方法依旧可行，代码：

```javascript
// 第一个 setTimeout 用“.bind()”方法修改如下：
setTimeout(this.sayName.bind(this), 1000)
```

### “匿名函数包裹”方法

“匿名函数包裹”方法好像让事情更复杂了——修改后但**没能解决问题**的代码如下：

```javascript
// 第二个 setTimeout 用“匿名函数包裹”方法修改如下：
setTimeout(function(){
    this.sayName()
}, 1000)
```

此时，`this.sayName` 的值为 `undefined`，`sayName` 方法甚至都没有机会得到调用。

#### “匿名函数包裹+`.bind`”

在包裹匿名函数后的代码的基础上，仍可以使用 `.bind` 方法。

```javascript
// 第二个 setTimeout 用“匿名函数包裹+.bind”方法修改如下：
setTimeout(function(){
    this.sayName()
}.bind(this), 1000)
```

#### “匿名函数包裹+`that`”

可以利用匿名函数形成闭包，引用正确的 `this`：

```javascript
// 第二个 setTimeout 用“匿名函数包裹+that”方法修改如下：
var that = this
setTimeout(function(){
    that.sayName()
}, 1000)
```

### “箭头函数”方法

ES6 的箭头函数没有自己的 `this` 、直接继承外部作用域的 `this` ，所以还可以这么改：

```javascript
// 第二个 setTimeout 用“箭头函数”方法修改如下：
setTimeout(()=>{this.sayName()}, 1000)
```

## 小结

当 `setTimeout` 的回调函数中出现 `this` 的时候，要特别注意其绑定的对象是否和预想的一致。当绑定有误时可以通过下述方法解决。

当 `setTimeout` 的回调函数是一个能够通过变量引用的对象的方法（类似于例子中的 `imo.sayName`）时，有两种解决方法：

1. “`.bind()`”方法
2. “匿名函数包裹”方法

当 `setTimeout` 的回调函数是一个通过 `this` 访问的方法（类似于例子中的 `this.sayName`）时，有两种解决方法：

1. “`.bind()`”方法
2. “ES6箭头函数”方法

当 `setTimeout` 的回调函数是一个含有 `this` 的匿名函数（类似于例子中的 `this.sayName` 被匿名函数包裹后）时，有两种解决方法：

1. “`.bind()`”方法
2. “闭包that”方法

> 可以看出，解决方法中的“`.bind()`”方法是万金油。所以，如果你记不住这么多方法，至少也要记住“`.bind()`”方法。
