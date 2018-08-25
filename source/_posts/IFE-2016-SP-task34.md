---
title: IFE-2016-SP-task34 Square Following Directives II
date: 2017-01-08 20:57:49
tags: [IFE]
---

## My Solution

My solution to task34 is available on jsfiddle: 
{% jsfiddle yo8udddy 'default' 'light' '100%' '400px' %}

<!-- more -->

## Compared with others'

### [任务得分第一的团队的 solution](http://ife.baidu.com/2016/review/detail?workId=2118)

* :ribbon: 判断对象是否含有某属性 
    
    ```javascript
    // 我也不记得自己当初为何脑抽写得这么繁琐了。。:/
    var getValidDirective = function(rawValue) {
      var possibleValue = [];
      for (var key in square) {
        possibleValue.push(key);
      }
      // 为“判断对象是否含有某属性”，我啰哩啰嗦地写了上面一大堆外加下面的 if 判断条件 :/
      var value = rawValue.trim().replace(/\s/g, '').toLowerCase();
      if (possibleValue.indexOf(value) > -1) {
        return value;
      }
      throw new Error("非法指令，请重新尝试 :(");
    };

    /* 化简如下：*/

    var getValidDirective = function(rawValue) {
      var value = rawValue.trim().replace(/\s/g, '').toLowerCase();
      if (square[value]) {// 现在“判断对象是否含有某属性”化简为仅仅一行代码
        return value;
      }
      throw new Error("非法指令，请重新尝试 :(");
    };
    ```

* :no_mouth: [其他见 task33 时和该团队的比较](http://baishusama.github.io/2017/01/07/IFE-2016-SP-task33/#任务得分第三的团队的-solution)

### [任务得分第二的团队的 solution](http://ife.baidu.com/2016/review/detail?workId=8025)

* :mask: 评论说：“按照 task34 题意，（在四个死角，） MOV 指令执行失败的时候，应该既不旋转也不前进。”
    - [x] ToDo: 改进 MOV 相关逻辑

## 做 task34 时遇到的个别难点

大家 task34 的完成度都不是很高的样子。。我这里对我做 task34 的时候遇到的最难点，进行一下说明。

### 方向变量

我的小方块的方向 `sqDir` (squareDirection 的缩写）的定义借鉴了 css 中 top, right, bottom, left 的书写顺序。这里我规定：默认朝上的时候，`sqDir` 的值为 0 ，每顺时针旋转 90 度， `sqDir` 值加一，逆时针则减一。理论上，`sqDir` 的值可以为所有整数（当然是在 JS 数字范围内）。遵循这种规定的 `sqDir` 值能很好地对应到（只需乘以 90 便能得到） `transform` 属性的 `rotate` 的角度值。`sqDir` 的部分值的含义及和 `rotate` 角度的对应如下表：

| ... |  -4 |  -3 |  -2 |  -1 |**0**|**1**|**2**|**3**|  4  | ... |
|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|
| …… | 朝上 | 朝右 | 朝下 | 朝左 |**朝上**|**朝右**|**朝下**|**朝左**| 朝上 | …… |
|...|-360deg|-270deg|-180deg|-90deg|0deg|90deg|180deg|270deg|360deg| ... |

### 朝向何方

那么如何判定一个小方块是朝向什么方向的呢？

这里，我们需要数学含义上的取模运算，而不是 JavaScript 中的 `%` 运算。取模和取余的差别，在[上一篇](http://baishusama.github.io/2017/01/07/IFE-2016-SP-task33/)中提到过，这里也就不再多说了。于是，我定义了如下取模函数，用来将 `sqDir` 的值映射到 `0,1,2,3` 以便判断方向：

```javascript
// 真正的取模，而不是取余
var directionMod4 = function(direction) {
  return (direction % 4 + 4) % 4;
};
```

### 就近转向（往哪边转）

以上对于 `GO`、`TUN` 和 `TRA` 三种指令已经足够用了，但是 `MOV` 指令能任性地转向！然而，我不想让自己的小方块显得很蠢，多做一些无用功——转多余的角度才转到指定方向。

为了使小方块显得更加聪明，我们需要就 `0,1,2,3` （即往上转，往右转，往下转，往左转）这四种输入找到小方块在当前朝向（`directionMod4(sqDir)`）下，往哪边（顺时针还是逆时针）转向能最快地转到位。

这里最好的办法应当是能找到一个类似循环链表的数据结构，然后同时向左和向右查找，类似深度优先地返回最
短路径——所转角度最小的转向。

可惜我没有找到理想的数据结构，就用了两个 `while` 循环来实现上述逻辑，代码如下：

```javascript
// 返回转到某方向的最小角度
var turnTo = function(direction) {
  var curDir = directionMod4(sqDir);// 小方块的当前朝向
  // if(curDir === direction) return 0;

  /* 就近转向 */
  var cwStep = 0,// 顺时针（clockwise）所需转向次数
    anticwStep = 0;// 逆时针（anticlockwise）所需转向次数
  while (true) {
    if ((curDir + cwStep) % 4 === direction) {
      break;
    }
    cwStep++;
  }
  while (true) {
    if ((curDir - anticwStep + 4) % 4 === direction) {
      break;
    }
    anticwStep++;
  }

  var deg = 0;
  if (cwStep > anticwStep) {
    deg = -anticwStep * 90;
  } else {//if(cwStep <= anticwStep)
    deg = cwStep * 90;
  }
  return deg;
};
```

以上代码的注意点：

1. `cwStep` 和 `anticwStep` 分别代表`顺指针所需转向次数`和`逆时针所需转向次数`。两者都可以为零，为零的时候函数将返回 0 。
2. 为了将方向变化控制在 `0,1,2,3` 四个数值内，`while` 循环里的 `if` 判断条件中用了 `% 4` 运算，以避免超过 `3` ；计算逆时针旋转所需次数的时候，`curDir - anticwStep + 4` 中的 `+ 4` 避免小于 `0` 。
3. 当 `顺指针所需转向次数 === 逆时针所需转向次数` 时，统一采用顺时针转向。

再重新审视一下转向逻辑的话，不难发现：

1. `顺指针所需转向次数`和`逆时针所需转向次数`之和总是等于 4 的，即 `anticwStep === 4 - cwStep`。由此我们可以去掉负责计算 `anticwStep` 的第二个 `while` 循环。
2. `cwStep` 的值可能为 `0,1,2,3` 四个数（`anticwStep` 对应为 `3,2,1,0`），仅当 `cwStep` 的值为 3 的时候，逆时针方向转角才小于顺时针。

综上，上面的代码可以简化如下：

```javascript
// 返回转到某方向的最小角度
var turnTo = function(direction) {
  var curDir = directionMod4(sqDir);// 小方块的当前朝向
  // if(curDir === direction) return 0;

  /* 就近转向 */
  var cwStep = 0;// 顺时针（clockwise）所需转向次数
  while (true) {
    if ((curDir + cwStep) % 4 === direction) {
      break;
    }
    cwStep++;
  }

  var deg = 0;
  if (cwStep > 2) {
    deg = (cwStep - 4) * 90;
  } else {
    deg = cwStep * 90;
  }
  return deg;
};
```

## To Do:

- [ ] 就近转向的更好方式？

## 结语

最后献上，今天看的 S 校某公开课上讲师引用的一句话：

> "Really the world is just one big spectrum and if you go around far enough to one side, you end up on the other."