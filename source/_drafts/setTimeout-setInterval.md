---
title: setTimeout & setInterval
tags: [js]
---

## 语法 & 参数

```
setTimeout(func/code[, delay, param1, param2, ...]);
setInterval(func/code, delay[, param1, param2, ...]);
```

第一个参数可以是 `code` ，即以字符串形式表示的代码。由于其内部使用的是名声狼藉的 `eval` ——[欺骗词法作用域导致性能下降](https://github.com/getify/You-Dont-Know-JS/blob/master/scope%20%26%20closures/ch2.md#cheating-lexical)，因此不推荐使用。

第一个参数推荐使用回调函数形式。注意，由于回调函数将在全局作用域中执行的，故**回调函数的 `this` 将指向全局对象**，即使这个回调函数是某个对象的方法。如果想让回调函数的 `this` 指向回调函数所从属的那个对象，可以使用以下两种方法：

1. 匿名函数包裹函数调用；
2. bind/call/apply 。

```javascript
    var name = "global";

    function User(username) {
        this.name = username;
        this.sayName = function() {
            console.log(this.name);
        };
    }

    var user = new User("baishu");

    // 回调函数的 `this` 指向的是全局对象
    setTimeout(user.sayName, 1000); // "global"

    // Way 1. 匿名函数包裹函数调用
    setTimeout(function() {
        user.sayName();
    }, 1000); // "baishu"

    // Way 2. bind/call/apply
    setTimeout(user.sayName.bind(user), 1000); // "baishu"
```

第二个参数，是一个数字，表示毫秒。`setTimeout` 中该参数的最小值取到 0 ； `setInterval` 中该参数的最小值取到 10 。然而实际使用中，该参数的最小值可能更大。

情况一：嵌套多层的 `setTimeout` 。[HTML5 规范](https://html.spec.whatwg.org/multipage/webappapis.html#timers)中有提到：

> 8. If timeout is less than 0, then set timeout to 0.  
> 如果 timeout 小于 0 ，那么设置 timeout 为 0 。
> 
> 9. If nesting level is greater than 5, and timeout is less than 4, then set timeout to 4.  
> 如果嵌套层级大于 5 ，且 timeout 小于 4 ，那么设置 timeout 为 4 。

第三个参数开始的参数，是回调函数的参数。IE9 及以下只支持前两个参数，可以通过：

1. 匿名函数包裹带参数的函数调用；
2. bind/call/apply 。

```javascript
// Way 1. 匿名函数包裹带参数的函数调用
setTimeout(function() {
    myFunc(param1, param2, ...);
}, 1000);

// Way 2. 匿名函数包裹带参数的函数调用
setTimeout( function(param1, param2, ...){}.bind(undefined, param1, param2, ...), 1000 );
```

两种方式解决上述兼容性问题。

## 机制

### 知识储备

[JavaScript has a concurrency model based on an "event loop".](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/EventLoop)

JavaScript 是单线程的（HTML5 提出的 Web Worker）。

JS 有三个主线程，队列（event loop）、GUI 渲染线程、。。。？？？



一个 JavaScript 运行时（runtime）包含三部分：

* 一个函数调用组成的堆栈帧（a stack of frames）。即[《[翻译] JavaScript 事件循环：说明》](http://baishusama.github.io/2017/04/13/翻译-The-JavaScript-Event-Loop-Explained/)中提到的调用栈（call stack）。
* 所有对象被分配到的一个堆（heap）。
* 一个待处理的消息队列（message queue）。

而事件循环（event loop）的工作就是时刻注意（？？？还是空闲时注意？？？）上面说到的“栈”和“队列”的情况（是否为空）。如果“栈”为空，且“队列”中有消息，（由于队列总是先进先出的，）取第一条消息，其回调函数的调用将作为“栈”的初始帧，[并且由于 JavaScript 是单线程的，因此在堆栈上返回所有调用之前暂停进一步的消息的轮询和处理。](http://baishusama.github.io/2017/04/13/翻译-The-JavaScript-Event-Loop-Explained/)

事件循环之所以称为“事件循环”，是因为其实现方式多半是类似如下的形式：

```javascript
while(queue.waitForMessage()){
  queue.processNextMessage();
}
```

### 机制

了解了以上相关知识后，我们就可以一探定时器的机制了。

当执行中的代码，遇到 setTimeout/setInterval 时，会将相关代码入栈，然后立即出栈，并将计时任务转交给浏览器的 Web APIs 计时。调用定时器的代码继续同步（sync）执行。当计时完毕， Web APIs 会将定时器的回调函数作为消息，进入消息队列，等待处理。当同步代码执行完毕，调用栈（stack）为空时，事件循环（event loop）会依次从消息队列中取出消息逐个先后执行。



特殊的，有 `setTimeout(callback, 0);` 。其机制。。。

## `setInterval` 的缺点

## IMPORTANT

我其实比较想说的是，我的一些发现：

1. alert 阻塞 setInterval ，阻塞终止后，setInterval 仍按照原来的时间间隔执行——点击 alert 的“确定”按钮后，如果原来的 delay 是 5s，那么 5s 后才会有下一个 alert 。
    * 猜测：alert 造成阻塞，而 js 是单线程的，无法继续进行计时等操作，故阻塞消失之后，计时重新从头开始。
    * 猜测：出于性能考虑，消息队列中，同一个（不同的可以？？？）定时器的消息最多只能出现一次，即 web API 会判断，队列中是否已经有某个定时器的回调函数的消息了，如果已经有了，就不进入队列。且等到，上一个消息结束，web API 会重新从头开始计时？？？
2. 事件执行顺序。js 模仿用户点击事件的 click() 的回调函数被视为同步的了。而用户点击触发的一般总是异步的？？？

## 应用

1. 调整事件的发生顺序
    * 子元素的事件回调函数会早于父元素的事件回调函数。
    * 用户自定义的回调函数通常在浏览器的默认动作之前触发。e.g. 用户在输入框输入文本，keypress事件会在浏览器接收文本之前触发。
2. 分片
3. 防抖（debounce）

## 如何适合使用？？？

## 参考

* [你所不知道的 setTimeout](http://jeffjade.com/2016/01/10/2016-01-10-javacript-setTimeout/)：一些说法存在疏漏。
* [你所不知道的 setInterval](http://jeffjade.com/2016/01/10/2016-01-10-javaScript-setInterval/)：一些说法存在疏漏。
* [setTimeout @MDN](https://developer.mozilla.org/en-US/docs/Web/API/WindowOrWorkerGlobalScope/setTimeout)
* [setInterval @MDN](https://developer.mozilla.org/en-US/docs/Web/API/WindowOrWorkerGlobalScope/setInterval)
