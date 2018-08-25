---
title: IFE-2016-SP-task35 Square Following Directives III
date: 2017-01-20 12:15:42
tags: [IFE]
---

## My Solution

My solution to task35 is available on jsfiddle: 
{% jsfiddle 0oesrpwL 'default' 'light' '100%' '500px' %}

<!-- more -->

### :bookmark: `&&` 和 `||` 的使用方法

#### [Logical Operators @MDN](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Logical_Operators)

`&&` 会返回第一个假值（falsy value），`||` 会返回第一个真值（truthy value）。假值包括：

1. `undefined`
2. `null`
3. `0`
4. `NaN`
5. `""`

```javascript
/* 下面两个函数等价 */
function shortCircuitEvaluation() {
  doSomething() || doSomethingElse()
}

function equivalentEvaluation() {
  var flag = doSomething();
  if (!flag) {
    doSomethingElse();
  }
}
/* 一些例子 */
"" && false      // returns ""
"" || false      // returns false
false || ""      // returns ""
```

#### [对 js 运算符 || 和 && 的总结](http://jianguang-qq.iteye.com/blog/462449?page=2)

```javascript
/* e.g.1 成长速度 */
// 需求一（==） - basic:
var addLevel = (addStep==5 && 1) || (addStep==10 && 2) || (addStep==12 && 3) || (addStep==15 && 4) || 0;
// 需求一（==） - with Object:
var addLevel={'5':1,'10':2,'12':3,'15':4}[addStep] || 0;      
// 需求二（>）:
var addLevel = (addStep>12 && 4) || (addStep>10 && 3) || (addStep>5 && 2) || (addStep>0 && 1) || 0;

/* e.g.2 jQuery 源码 */
var wrap = !tags.indexOf("<opt") && ...;// means 'startWith'
```

使用注意点：

* 优点：精简了代码，能减少网络流量
* 缺点：降低了代码的可读性
* 推荐做法：适当写些注释

#### 我在自己的实现中的使用
```javascript
// 前情提要：directive will be "go","tunlef2",...

var temp = directive.match(/\d+/) && directive.match(/\d+/)[0]; 
// e.g. null && ~ -> null | ["1"] && "1" -> "1"

step = parseInt(temp) || 0; 
// 当值为 null 时，得到 NaN 。NaN || 0 -> 0

// 以及其他很多的 '... || 默认值'
```

### :ribbon: textarea 避开监听滚动事件来实现同步滚动行号

#### 起因

> 因为，做之前稍微看了眼各个团队得到的评价（评论），了解了[导师对这个任务实现的一些看法](http://ife.baidu.com/2016/review/detail?workId=8467)，其中第四条指出：“指令输入编辑器显示代码行逻辑实现上有点不优雅。 **一定要监听滚动才能实现左边指令行数一起滚动吗？** 包括每次输入变化都重新渲染所有的代码行数，代价有点大。”
> 
> 于是，我花了很久来思考怎么才能避开我一开始预想的通过监听滚动来实现行号和内容的同步滚动。最终得到了这个实现。

#### 思路

在一个父级 `div` 中两个同级的 div 正常情况下能同步地上下滚动。

所以我们需使 `textarea` 表现得和普通的 `div` 一样——具有和 `div` 一样的特性——能够根据所包含的文字内容的高度，自动地变化自己的高度。

#### 实现

##### textarea 高度自适应的问题 - 法一

> **参考：**
> [Autosizing textarea using Prototype @SO](http://stackoverflow.com/questions/7477/autosizing-textarea-using-prototype) 提问中 Jan Miksovsky 的回答，以及回答下面的评论提出的一些修改意见。

可以查看 [`jsfiddle` 上的在线 demo]()。

这个解决方法 **棒** 在更多地依赖 HTML 和 CSS，而不是 JS 。

但是也存在 **缺点** ：在最后一行进行回车的时候，高度会有波动，而不是直接一步到位。

**根本原因** 在于，回车的一瞬间 `textarea` 的高度还未发生变化。因为在本例中，`textarea` 的高度为 `height: 100%;` 是依赖于父元素的高度，而父元素的高度依赖于 `div#textCopy` 的高度。

其具体过程是这样的：在最后一行进行回车的时候，`textarea` 的文字内容的最下方会增高一行，此时父元素高度不变，即 `textarea` 的高度不变，默认地 `textarea` 内容高度大于元素本身高度、第一行的内容将上移并被遮挡。而回车触发了 `keydown/keyup` 事件，回车被 JS 中定义的 `autoSize()` 转换成了 `<br/>` 并传给了 `div#textCopy` ，`div#textCopy` 的高度由此变高，父元素和 `textarea` 的高度也相应变高。第一行的内容也就由此能够最终避免被遮挡的命运了。

##### textarea 高度自适应的问题 - 其它方法

> 其它待读的相关资料：
> [Creating a textarea with auto-resize](https://stackoverflow.com/questions/454202/creating-a-textarea-with-auto-resize)
> 
> 可以适当参考其它成熟实现：
> [babeljs.io](https://babeljs.io/repl/)

#### 小结

由于目前需要实现的并不是多么完备而成熟的文本编辑器（需求详见 [task35 任务说明](http://ife.baidu.com/2016/task/detail?taskId=35)），本次实现的文本编辑器还存在若干有待改进之处。

[ ] **To Do**

1. [ ] 默认命令代码在一行内——未考虑到，当输入一行很长的代码，且不换行的情况下，行号间应该存在空隔。
2. [ ] 默认命令代码的数量是很有限的——未考虑到，当输入行数很多的情况下，数字很大的行号的样式问题。
3. [ ] 默认用户通过键盘来键入命令行——未考虑除了键盘输入外的其他输入方式（鼠标右键复制等）。
4. （有待补充。也强烈欢迎各位看官提出 bug ~）

## Compared with others'

### [任务得分第一的团队的 solution](http://ife.baidu.com/2016/review/detail?workId=2417)

#### 一些实现上的差异比较

* `setTimeout` 还是 `setInterval`
    - 我的指令逐行执行的实现：`for` 循环 + `setTimeout` + 闭包。
    - 该团队的实现：`setInterval` + 其外部的计数器。
        + 优点：
            1. 省去了 `setTimeout` 第二个时间参数的计算。
            2. 由于自带重复，省去了外层的 `for` 循环。
            3. 因而也不需要闭包。
            4. 在终止后续指令的时候，只需要 `clearInterval` 当前即可。而 `setTimeout` 需要 `clearTimeout` 后续所有。
* 判断指令是否合法
    - 我是通过对象的方法是否存在判断的。e.g. `!square[name]` 表示方法不存在时。
    - 该团队 `for in` 所有方法名得到正确指令的正则 `consoleExp` 再用正则判断的。

### [任务得分第二的团队的 solution](http://ife.baidu.com/2016/review/detail?workId=3569)

~~P.S. 这位的代码我没细看，但是这位的头像和（aiti？）一样，和我企鹅群里认识的那人是同一个人么OAO？。。~~

## 对 task36 的展望 ~~（恐慌）~~

~~看了眼 task36 前三的团队实现的 demo 和得到的评价，我觉得我可能要犯拖延了。。第一名的团队支持图片上传。。前三的某支团队甚至搞了一个 AST 。。大神们，请收下在下的膝盖 Orz 。。。~~

在看了[一篇文章](https://zhuanlan.zhihu.com/p/22213177)的“害怕开展新项目怎么办？”“不要完美主义”的部分内容之后，决定还是克服拖延开始这个系列任务的最后一个吧。加油！！

并且在原作者[另一篇文章](https://medium.freecodecamp.com/learning-to-code-when-it-gets-dark-e485edfb58fd#.t2ecehmpn)中得到了肯定：

> The thing I am sure about is — **if a person stays with coding long enough and makes the practice deliberate — they will get to any level they aspire to.**
> 
> By **deliberate practice**, I mean:
> 
> * examining the bugs and problems in the code
> * going back to certain problems and trying to solve them in a better way
> * reading other people’s code to see how they solved these problems
> * refactoring your old code
> 
> To put it simply: [doing rework rather than trying new things and switching topics all the time](http://jamesclear.com/stay-on-the-bus) [ ] .

