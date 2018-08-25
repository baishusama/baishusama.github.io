---
title: Three Column Layouts Cheat Sheet
date: 2017-04-21 22:24:46
tags: [CSS]
---

整理了下常见三栏布局的做法。

## 布局索引

1. [流式布局](https://baishusama.github.io/stockyard/three-column-layout.html#1-流体布局)
2. [BFC 布局](https://baishusama.github.io/stockyard/three-column-layout.html#2-BFC布局)
3. [绝对定位布局](https://baishusama.github.io/stockyard/three-column-layout.html#3-绝对定位布局)
4. [圣杯布局](https://baishusama.github.io/stockyard/three-column-layout.html#4-圣杯布局)
5. [双飞翼布局](https://baishusama.github.io/stockyard/three-column-layout.html#5-双飞翼布局)
6. [Flex 布局](https://baishusama.github.io/stockyard/three-column-layout.html#6-flex布局)

<!-- more -->

> P.S. 各个布局的原理在本篇没有解释，对于圣杯布局和双飞翼布局感兴趣的童鞋可以看看参考中的后两篇文章。

## 优缺点

前两种布局的缺点是主要内容无法优先加载。当页面加载慢时会影响用户体验。

后四种布局调整了 HTML 代码顺序，克服了上述缺点。

与圣杯布局相比，双飞翼布局以更复杂的 HTML （主要内容外再包裹一层）为代价简化了 CSS 样式（左侧边栏少用一个相对定位）。

## 参考

* [详解 CSS 七种三栏布局技巧 @知乎](https://zhuanlan.zhihu.com/p/25070186?refer=learncoding)
* [圣杯布局](http://chen106106.iteye.com/blog/1631865)
* [双飞翼布局介绍-始于淘宝UED](http://www.imooc.com/wenda/detail/254035)