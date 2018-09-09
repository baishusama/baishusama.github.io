---
title: '[小程序] 复用下拉刷新、上拉加载页面配置'
date: 2018-09-08 17:01:46
tags: ['微信小程序']
---

## 前言

之前使用 Ionic 开发公司的 app 的时候，写过通过 TS 的 `class` 的 `extends` 来达到复用目的的代码，减少了很多工作量，维护起来也更省心。

这两周开始开发小程序，出于几个原因我只使用了小程序的原生开发能力：

- 因为是第一个小程序项目，想 **体验一下小程序的原生开发** 过程
- 有两个很优秀的第三方框架——[wepy](https://github.com/Tencent/wepy) 和 [mpvue](https://github.com/Meituan-Dianping/mpvue) 但我对这两个 **框架选择困难**。也只有开发过原生、了解原生开发的痛点所在之后，才能有理有据、有针对性地选择某一个框架。
- 而且引入第三方框架存在风险，也许屏蔽了一些原生开发的坑，但是 **框架也可能引入新的坑**……
- 这个小程序 **需求比较简单**，原生小程序开发理应 hold 住

然后我又碰到了两个大部分逻辑（下拉刷新、上拉加载）相似的列表页。还是出于“懒惰“，我想尽可能复用代码。

凭着刷过一遍文档的模糊印象，我本来以为小程序提供的 `Behaviors` 是为这个场景量身定制的，然后重看文档才发现 [`Behaviors` 仅支持 `Component` 的复用](https://developers.weixin.qq.com/miniprogram/dev/framework/custom-component/behaviors.html)……

所以要怎么复用 `Page` 的相关代码呢？因为[小程序原生支持绝大多数 ES6 的 api](https://developers.weixin.qq.com/miniprogram/dev/devtools/details.html#%E5%AE%A2%E6%88%B7%E7%AB%AFes6-api-%E6%94%AF%E6%8C%81%E6%83%85%E5%86%B5)，我自然而然就想到了用 ES6 的 `class` 和 `extends` 的一套来实现！于是我掉进了坑里……

<!-- more -->

## 尝试一：ES6 的 `class` 和 `extends`，以失败告终

等我兴高采烈地把需要复用的代码放到一个 `UnlimitedListAbstractPageConfig` 类中，然后在页面代码中用子类 `ProductListPageConfig` 继承并实例化后作为参数传入 `Page` 后我发现页面没有效果：

```javascript
/**
 * 我最初想当然的代码
 */

// unlimited-list.abstract.js
class UnlimitedListAbstractPageConfig {
  constructor(){
    // this.paginationService = new PaginationService();
  }
  onLoad(){
    // fetch data on page load
  }
  onReachBottom(){
    // fetch and append new data to data list
  }
}
module.exports = UnlimitedListAbstractPageParams;

// product-list.js
const UnlimitedListAbstractPageConfig = require('../unlimited-list.abstract');

class ProductListPageConfig extends UnlimitedListAbstractPageConfig {
  constructor() {
    super();
    this.data = {
      // necessary data
    };
  }

  bindProductTap(){
    // go to product retrieve page
  }
}

const pageConfig = new ProductListPageConfig();
Page(pageConfig);
```

没有效果，控制台也没有报错。发现不管是父类中定义的方法还是子类中定义的方法都没有执行，但是属性（e.g. `data`）都能正常访问和显示。通过在控制台打印 `pageConfig` 和翻了下[《YDKJS》的相关章节](https://github.com/getify/You-Dont-Know-JS/blob/master/es6%20%26%20beyond/ch3.md#class)我才了解到 ES6 的 `class` 的方法是不可枚举的。

我猜测问题可能是出在了这里。因为[小程序官方文档](https://developers.weixin.qq.com/miniprogram/dev/framework/app-service/page.html)中有这么一句话：

> Object 内容在页面加载时会进行一次深拷贝，需考虑数据大小对页面加载的开销

关键词是“拷贝”，拷贝的仅仅是可枚举的属性么？

带着疑惑我去翻了一下 `Page` 的源码：

1. 小程序开发工具控制台面板的 Sources 找到并下载 WXService.js 文件（可以控制台打印 `Page` 并双击结果快速定位或者手动找到 `appservice/__dev__/` 目录）
2. 因为是压缩过的 JS 代码，先格式化一下（可以通过控制台的“{}”按钮、编辑器、在线工具等等）
3. 借助 [beautify with words](https://github.com/zertosh/beautify-with-words) 工具，将被压缩过、难以全局搜索的变量名变长，从而利于全局搜索

尽管如此，还是比较费劲地才找到了大概率是“罪魁祸首”的代码：

```javascript
// ...
(scoduren.Page = function prutupe(
  cocile,
  culade,
  maqueker,
  vugreeme
) {
  var jefatey = vugreeme || __wxAppCurrentFile__;
  if (!jefatey)
    return void console.error(
      'Page constructors should be called while initialization. A constructor call has been ignored.'
    );
  var bafeyad = maqueker || __wxAppCode__,
      glosuhi = Object.create(null),
      amanuk = Object.create(null);
  /**
   * ImoNote: 注意这里的 for..in ！！
   * for..in 遍历的是可枚举属性，
   * 从而 es6 的 class 里定义的 methods 由于不可枚举所以没有效果……
   */
  for (var himamum in cocile)
    'data' !== himamum &&
    ('function' == typeof cocile[himamum]
      ? (amanuk[himamum] = cocile[himamum])
      : (glosuhi[himamum] = cocile[himamum]));
  // ...
}
// ...
```

嗯，找到原因之后，我只能放弃用 ES6 的 class 来实现了，因为根本实现不了……

## 尝试二：写个工厂函数，自己动手丰衣足食

思路很简单，就是写个生成对象的工厂函数，因为对象的属性和方法（除非手动指定）都是可枚举的：

```javascript
// 工厂函数
const UnlimitedListAbstractFactory = function(){
  // this.paginationService = new PaginationService();
  
  const pageConfig = {
    onLoad(){
      // fetch data on page load
    }
    onReachBottom(){
      // fetch and append new data to data list
    }
  };

  return pageConfig;
}

module.exports = UnlimitedListAbstractFactory;
```

上述代码是对照第一版（ES6 class 的那版）写的，可能看上去有点乱，本质就是：

```javascript
// 工厂函数的骨架
const factory = function(){
  const o = {};
  return o;
}
module.exports = factory;
```

就这么简单！

我们通过工厂函数来快速生成需要的对象。在复用页面配置对象的这个场景里，我们将会在这个对象里，定义我们多个页面共用的代码（e.g. `onLoad`, `onReachBottom`...）。这样我们就不用在多个不同的页面分别编写和维护这部分代码，以达到代码复用的目的。

> TODO:【到底要不要贴全部的代码？一点一点地贴代码更好么？……】
>
> - 此时调用方的代码如何？
> - 以及代码可以借助 `prototype` 进一步优化？

## 附录

### 附录A. 本文相关的核心代码 DEMO

> TODO: 准备写个小程序 demo（敬请期待。。）

### 附录B. 我在 Ionic app 中实现的继承关系的罗列

文章最初提到过的 Ionic 的 app 中的继承关系如下：

- ListTpl
  - ProductListTpl
    - HomePage
    - ShortProductListPage
    - DpFixincomeListPage
    - DpStockListPage
  - NewsListPage
  - ProgressPage
- RetrieveTpl
  - ShortProductRetrievePage
  - DetailProductRetrievePage
  - NewsRetrievePage
  - StatisticsPage

其中，以 `*Tpl` 都是 `abstract class`，所有 `*Page` 都是 `extends` 抽象类的具体的类实现。