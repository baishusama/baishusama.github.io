---
title: 使用 JavaScript 操作 HTML 批量制作 Anki 卡片
date: 2017-02-27 11:44:54
tags: [anki,markdown,sublime,HTML,JavaScript]
---

## 想要达到的效果

1. 使用 markdown 总结笔记
2. 利用笔记批量生成 anki 卡片

## 前期准备

### 批量制作 anki 卡片的两个思路

1. [利用 TinyTask 之类的自动化操作软件进行重复性操作](http://v.ku6.com/show/l1A1FiYXohYYsz-OM6RdyQ...html)；
2. 利用脚本对遵循某种规则的笔记内容进行切分和生成卡片。

就这两个思路来说，第二个思路一劳永逸。而且作为一个程序猿，闭着眼睛也会选第二个的嘛 /w\。

<!-- more -->

### 我的工作流和“原料”

我一般是在 Sublime Text 上用 `markdown` 记各种笔记，然后通过 Sublime 的 OmniMarkupPreviewer 插件（在编辑页面 -> 右键 -> Preview Markup in Browser）在浏览器中实时预览 markdown 得到的 `html` 的。

简而言之， **`MD + ST(OmniMarkupPreviewer) => HTML`** 。

于是现在，我的手头有两种“原料”：

* 最开始的 **MD 文件**
* 实时的 **HTML 文件**

### 脚本语言的选择

看到 .html 文件，就想到了 `JavaScript`。所以，我选择使用 .html 文件作为“原材料”，用 js 对其进行加工，得到制作 anki 卡片所需要的 .csv 文件。

### 约定

> P.S. 这里简单起见，只制作具有正反面的、静态的（没有完形填空等的） ~~、纯文字内容的（没有图片、音频的）~~ 卡片。不过，卡片可以包含图片和表格。

思路中提到的 **“遵循某种规则”** 是指书写 .md 文件的时候，你需要想好用什么特殊的符号区分开卡片的正面和背面。

这里，我在 markdown 中**使用第二级标题 `##`** 来表示卡片的正面，在 HTML 对应为 `h2` 标签；而在两个第二级标题之间的所有内容表示卡片的反面。

## 代码

### [generateAnkiCards.js](https://github.com/baishusama/stockyard/blob/master/anki/generateAnkiCards.js)

调用 generateAnkiCards.js 的代码：

`(function() { var source = "https://baishusama.github.io/stockyard/anki/generateAnkiCards.js"; var s = document.createElement("script"); s.src = source; document.body.appendChild(s); })()`

* 使用方法一：在实时预览 MD 文件的浏览器标签页的地址栏中键入 `javascript:` ，再在后面添加上述代码，并回车。
* 使用方法二：打开控制台（OSX 下 `Cmd+Alt+I` ，Windows 下 `F12`），在控制台输入上述代码并回车。

注意：（推测 Chrome 可能出于安全考虑？），直接在 Chrome 地址栏中复制带有 `javascript:` 前导的代码，`javascript:` 会消失。

我这里，为了进一步偷懒，利用了 Mac 下的 aText 软件，为（包含 `javascript:` 前缀的）一行代码定义了 `js:anki` 缩写。以后每次使用的时候，只要在地址栏敲入 `js:anki` 并回车就可以了。

![使用 aText 定义缩写](http://ohz4k75du.bkt.clouddn.com/anki/aText-anki.jpg)

> P.S. 
> 如果你没有 aText 软件，但是你也想偷懒，那么你可以另一种更为通用的办法：把上述一行代码保存为浏览器的书签。然后需要的时候，鼠标轻轻一点就好～
> 进一步偷懒，可以使用 Chrome 上的 Vimium 插件。在预览标签页直接键入 `b` ，然后找到之前保存的书签，回车。

### [css](https://baishusama.github.io/stockyard/anki/anki-card.css) （可选）

使用方法：anki 主面板 -> 浏览 -> 需要应用样式的记忆库 -> 卡片 -> “格式刷”下方输入框内粘贴 css 样式。

## 使用流程

1. 在 Sublime Text 中使**用 markdown 记笔记**。使用 `##` 表示问题（卡片正面），后面紧接着写回答（卡片反面）。并利用 OmniMarkupPreviewer 插件在浏览器中实时预览。
2. 记完笔记之后，在浏览器相应标签页的地址栏中，**键入 `js:anki` 并回车保存** .csv 文件。
3. 将保存的 **.csv 文件导入 anki 记忆库**。
    1. 主界面，在某记忆库下，单击“导入文件”。![主界面，在某记忆库下，单击“导入文件”。](http://ohz4k75du.bkt.clouddn.com/anki/anki-import-step1.jpg)
    2. 选中上一步生成的 .csv 文件，单击“打开”。![选中上一步生成的 .csv 文件，单击“打开”。](http://ohz4k75du.bkt.clouddn.com/anki/anki-import-step2.jpg)
    3. 下拉框选择“相同时更新已有”，单选框选中“允许在字段中使用HTML”，单击“导入”。![下拉框选择“相同时更新已有”，单选框选中“允许在字段中使用HTML”，单击“导入”。](http://ohz4k75du.bkt.clouddn.com/anki/anki-import-step3.jpg)


## 副作用和不足

### 副作用

笔记里所有的半角双引号 `"` 都会被 generateAnkiCards.js 文件强制转换为半角单引号 `'` 。因为生成 .csv 文件时，`"` 被用于区分 excel 的列。

### 不足

1. 现在的 js 代码仅仅是“能用”。各方面来看都不是好的代码。
2. 目前，js 代码是用原生 js 实现的。后来才发现，用 OmniMarkupPreviewer 生成 html 的时候，页面中已经自动引入了 jQuery ——可以使用 jq 进一步简化代码。
3. css 代码为了图快，是复制粘贴加微调，所以目测有很多无用的代码、重复的代码。

## 参考

> [How to generate Flashcards for Ankidroid out of Markdown in Jekyll using Javascript](http://pascalwhoop.github.io/technology/2016/01/27/How-to-generate-Flashcards-for-Ankidroid-out-of-Markdown-in-Jekyll.html)
> [Hotlink resources like JavaScript files directly from GitHub @StackOverFlow](http://stackoverflow.com/questions/20311271/hotlink-resources-like-javascript-files-directly-from-github/20311329#24720957)