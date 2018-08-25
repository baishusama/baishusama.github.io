---
title: IFE-2016-SP-task17 AQI Histogram
date: 2016-12-24 17:19:46
tags: [IFE]
---

## My Solution

My solution to task17 is available on jsfiddle: 
{% jsfiddle 7txop9gL 'default' 'light' '100%' '400px' %}

<!-- more -->

## Compared with others'

In [top-three team solutions](http://ife.baidu.com/task/detail?taskId=17) to this task, [the third one](http://ife.baidu.com/review/detail?workId=8045) is very similar to my solution except for [the code snippet](https://github.com/Phoebe-Perry/ife_baidu_2016/blob/gh-pages/second_phase/ife-baidu_task_17/task_17.js) below:

```javascript
//跨浏览器事件绑定
function addEventHandler(ele, event, hanlder) {
  if (ele.addEventListener) {
    ele.addEventListener(event, hanlder, false);
  } else if (ele.attachEvent) {
    ele.attachEvent("on"+event, hanlder);
  } else  {
    ele["on" + event] = hanlder;
  }
}
```

which also takes **browser compatibilities** into consideration. Note that: **`attachEvent` is for IE and `addEventListener` is for the others**.

## To Do

- [ ]  Plus, the top-two solution's UI and animation (or interaction) are very nice and worthwhile **to do** study in the near future.
- [ ]  Further, the part judging if some days are in a week/month is not a general way due to dates in data are continuous specially.