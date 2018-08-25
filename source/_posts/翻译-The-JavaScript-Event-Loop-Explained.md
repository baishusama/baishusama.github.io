---
title: '[翻译] JavaScript 事件循环：说明'
tags:
  - javascript
date: 2017-04-13 00:17:13
---


> 本文译自 Erin Swenson-Healey 的[The JavaScript Event Loop: Explained](http://blog.carbonfive.com/2013/10/27/the-javascript-event-loop-explained/)一文。

> 关键词：
> 
> * 事件循环 - The Event Loop
> * 运行环境 - runtime
> * 调用栈 - call stack
> 
> （译者注：“下面正文开始～”）

<!-- more -->

## 这篇文章是关于什么的？

## 这篇文章是写给谁的？

## 无阻塞的 I/O

（前三小节译者战略性省略）

## 事件循环（The Event Loop）

调用者（caller）从响应中的解耦是 JavaScript 运行环境能够在等待异步操作完成然后它们的回调执行的同时做一些别的事情的基础。但是，这些回调是保存在内存的哪里呢？它们是以怎样的顺序执行的呢？是什么导致它们被调用的呢？

JavaScript 运行环境包含一个消息队列（message queue），它存储了一长串待处理的消息以及这些消息相关的回调函数。当一个提供了回调函数的外部事件（例如，发生一个 click 事件或者接收到一个 HTTP 请求的响应）发生时，这些消息就会进入队列。假设一个用户点击了一个按钮，但这个按钮没有绑定任何的回调函数，那么将不会有消息进入队列。

在一次循环中，队列被轮询下一个消息（每一个轮询（poll）称为一个“列出（tick）”），若此时存在一个消息，那个消息对应的回调就会被执行。

![](http://blog.carbonfive.com/wp-content/uploads/2013/10/event-loop.png)

这个回调函数（译者注：指 init 函数）的调用在调用栈中是初始帧，并且由于 JavaScript 是单线程的，因此在堆栈上返回所有调用之前暂停进一步的消息的轮询和处理。后续（同步的）函数调用（译者注：指 link.addEventListener 函数）将新的调用帧添加到堆栈（~~例如，函数 init 调用了函数 changeColor~~（译者注：括号内的话感觉有误，故删去））。

```javascript
function init() {
  var link = document.getElementById("foo");

  link.addEventListener("click", function changeColor() {
    this.style.color = "burlywood";
  });
}

init();
```

在这个例子中，一个消息（和一个回调，即 changeColor）将在用户点击 foo 元素、触发一个 `onclick` 事件的时候进入队列。当这个消息出队列的时候，它的回调函数 changeColor 将被调用。当 changeColor 返回（或者抛出一个错误）的时候，事件循环继续。只要函数 changeColor 还存在，仍是 foo 元素的 onclick 的回调，后续在该元素上的点击会产生更多的消息（并和回调 changeColor 一起）进入队列。

## 队列中的附加消息

如果你的代码中的一个函数调用是异步的（像是 setTimeout），被提供的回调会在不久的将来的事件循环的某次列出（tick）时，作为一个不同的队列中的消息的一部分最终被执行。例如：

```javascript
function f() {
  console.log("foo");
  setTimeout(g, 0);
  console.log("baz");
  h();
}

function g() {
  console.log("bar");
}

function h() {
  console.log("blix");
}

f();
```

由于 setTimeout 的非阻塞特性，它的回调将在至少 0 毫秒的未来执行，而且不会作为本次消息的一部分被处理。在这个例子中，setTimeout 被调用了，传入了一个回调函数 g 和一个 0 毫秒的延迟。当这个指定的时间流逝后（在这个例子中，几乎是立即的），一个单独的消息包含 g 作为回调函数将会进入队列。控制台的结果将会是：“foo”，“baz”，“blix”，然后在事件循环的下一个列出（tick）会是：“bar”。如果在同一个调用帧，setTimeout 被调用了两次——且两次传递相同的参数值——它们的回调将在队列中保持调用时的顺序。

## Web Workers

使用 Web Workers 使你能够分流一个消耗高昂的操作到一个单独的线程去执行，从而解放主线程去做其他事情。workers 区别于初始化它的最初的线程，拥有它自己单独的消息队列、事件循环和独立的内存空间。worker 和主线程之间的沟通是通过消息传递实现的，这看起来很像传统的、事件的（evented）代码——我们已经见过示例。

![](http://blog.carbonfive.com/wp-content/uploads/2013/10/web-workers.png)

首先，我们的 worker：

```javascript
// 我们的 worker 做了一些 CPU 密集型操作
var reportResult = function(e) {
  pi = SomeLib.computePiToSpecifiedDecimals(e.data);
  postMessage(pi);
};

onmessage = reportResult;
```

然后，在我们的 HTML 中的 script 标签中存在的主代码块：

```javascript
// 我们的核心代码
var piWorker = new Worker("pi_calculator.js");
var logResult = function(e) {
  console.log("PI: " + e.data);
};

piWorker.addEventListener("message", logResult, false);
piWorker.postMessage(100000);
```

在这个例子中，主线程产生了一个 worker 并且为它的 message 事件注册了一个 logResult 回调函数。在这个 worker 中，它自己的 message 事件注册的事 reportResult 函数。当 worker 线程接收到一个来自主线程的 message 的时候，worker 将一个消息和对应的 reportResult 回调一起放入队列。当出列的时候，一个消息被回传给主线程，在主线程一个新消息连同 logResult 回调一起进入队列。在这种方式下，开发者能将 CPU 密集型操作委托给一个单独的线程，解放主线程去继续处理消息和事件。

## 关于闭包的一个提醒

JavaScript 对闭包的支持允许你注册这样的回调：当回调执行时，仍可以访问它们被创建时的环境，即使回调的执行创建了一个全新的调用栈。这很有趣：我们的回调是作为一个不同的消息（而不是它们被创建时的那个消息）的一部分被调用的。考虑如下例子：

```javascript
function changeHeaderDeferred() {
  var header = document.getElementById("header");
  
  setTimeout(function changeHeader() {
    header.style.color = "red";

    return false;
  }, 100);

  return false;
}

changeHeaderDeferred();
```

在这个例子中，changeHeaderDeferred 函数的执行中包含 header 变量。setTimeout 函数被调用，这导致了一个消息（附加 changeHeader 回调）在大约 100 毫秒的未来被添加到消息队列。changeHeaderDeferred 接着返回了 false ，结束了第一个消息的处理——但是 header 变量仍被一个闭包引用，因而没有被垃圾回收机制回收。当第二个消息被处理的时候，changeHeader 函数它仍能访问在该函数作用域外部被声明的 header 变量。一旦第二个消息（changeHeader 函数）处理完了，header 变量就能被垃圾回收机制回收了。

## 小贴士（Takeaways）

JavaScript 的 事件驱动的交互模型 和许多程序员习惯了的 请求-响应模型 不同——但就像你看到的，这并不是在造火箭。使用简单的消息队列和事件循环，JavaScript 使开发者可以围绕许多异步唤醒回调来构建系统，释放运行环境以处理并发操作，同时等待外部事件发生。但是，这只是并发的一种方式。在这篇文章的第二部分，我将比较 JavaScript 的并发模型和那些在 MRI Ruby（with threads and the GIL）、EventMachine（Ruby）、Java（threads）中的不同点。

## 拓展阅读

（略）













