---
title: Hexo 的 markdown-it 渲染引擎和其相关插件
date: 2016-12-24 21:17:01
tags: [hexo,markdown]
---

## 起因

我想在上一篇文章末尾的 to do 事项前面添加复选框，即想在 hexo 下使用 markdown 语法写文章时，使用 `[ ]` 、 `[x]` 这样 Github 上特有的 markdown 方言（Github Flavored Markdown, GFM）来输出 checkbox 。

在谷歌的过程中，当我看到 [github 上某位仁兄的抱怨](https://github.com/hexojs/hexo/issues/2161)说多种 hexo 的 markdown renderer 都对此问题无解的时候，一度陷入了绝望，甚至已经抱有了最终将使用非 markdown 语法（比如说，采用[使用 html 来实现 checkbox 的不太优雅的解决方案](http://zhoupq.com/%E7%94%A8-HTML-%E6%A0%87%E7%AD%BE%E5%AE%9E%E7%8E%B0-MarkDown-Task-List/)）的视死如归的觉悟。好在，暮然回首，[markdown-it](https://markdown-it.github.io/markdown-it/) 却在灯火阑珊处。

<!-- more -->

## 配置

1. 修改 hexo 的 markdown 渲染引擎：
    ```bash
    $ cd hexo-blog.github.io/ # 首先进入你的 hexo 的根目录
    $ npm un hexo-renderer-marked --save # 卸载 hexo 默认的 markdown 渲染引擎
    $ npm i hexo-renderer-markdown-it --save # 安装 markdown-it
    ```
2. 下载 markdown-it 的 markdown-it-checkbox 插件：
    ```bash
    $ cd node_modules/hexo-renderer-markdown-it/
    $ npm install markdown-it-checkbox --save
    ```
3. 在 hexo 的全局配置文件 `_config.yml` 添加以下：
    ```yaml
    # Markdown-it config
    ## Docs: https://github.com/celsomiranda/hexo-renderer-markdown-it/wiki
    markdown:
      render:
        html: true # Doesn't escape HTML content so the tags will appear as html.
        xhtmlOut: false # Parser will not produce XHTML compliant code.
        breaks: true # Parser produces `<br>` tags every time there is a line break in the source document.
        linkify: false # Returns text links as text.
        typographer: true # Substitution of common typographical elements will take place.
        quotes: '“”‘’' # "double" will be turned into “single”
                       # 'single' will be turned into ‘single’
      plugins:
        + markdown-it-abbr
        + markdown-it-checkbox # 本行启用了 checkbox 插件
        + markdown-it-emoji # 如果你想在 md 中使用 emoji 表情的话，需要另外下载相关插件
        + markdown-it-footnote
        + markdown-it-ins
        + markdown-it-sub
        + markdown-it-sup
      anchors:
        level: 2 # Minimum level for ID creation. (Ex. h2 to h6)
        collisionSuffix: 'v' # A suffix that is prepended to the number given if the ID is repeated.
        permalink: true # If true, creates an anchor tag with a permalink besides the heading.
        permalinkClass: header-anchor # Class used for the permalink anchor tag.
        permalinkSymbol: ¶ # The symbol used to make the permalink.
    ```
4. 重启 hexo 服务器，刷新 localhost 页面：
    ```bash
    $ hexo s -g
    ```

## 效果

最终，在文章中的显示效果：
- [ ] 本行前面的复选框是使用 `[ ]` 语法实现的
- [x] 本行前面的复选框是使用 `[x]` 语法实现的

## To Do - 待改进的遗留问题

### [x] 样式不太美观

* 用的是浏览器默认样式（体验不好），且不符合本文目前使用 NexT 主题风格。
* 书写时需要在 `[ ]` 后加两个空格，否则复选框和文字之间会没有空隙。

1. 找到自定义样式的文件：`themes/next/source/css/_custom/` 路径下的 `custom.styl` 。
2. 添加以下样式：
    ```css
    input[type="checkbox"] {
      display: none !important; // by imo: to overwrite styles in DuoShuo-Comments-Plugin
    }
    input[type="checkbox"] + label::before {
      content: '\a0';
      display: inline-block;
      margin-right: .2em;
      border: 1px solid;
      border-radius: .2em;
      width: .8em;
      height: .8em;
      vertical-align: .1em;
      text-indent: .1em;
      line-height: .7;
    }
    input[type="checkbox"]:checked + label::before {
      content: '\2713';
    }
    ```

### [ ] 非只读模式（可勾选和取消勾选）

### [x] 其它：使用 markdown-it 的 anchors 功能带来的副作用

- 使用 markdown-it 的 anchors 功能后，文章目录（Table of Content, TOC）中每个章节标题前均出现永久链接符号（默认为 `¶` ）。

1. 找到 hexo 定义 toc 函数的文件：`/node_modules/hexo/lib/plugins/helper` 目录下的 `toc.js` 。
2. 修改生成标题文本的代码行 `var text = $(this).text();` 为如下即可：
    ```javascript
    var text = $(this).text().slice(1);// by imo: remove markdown-it's anchor character in TOC
    ```

## 参考
* [配置 hexo-renderer-markdown-it](http://yangfch3.com/2016/05/08/hexo-experiences/hexo-renderer-marked-it.txt)
* ~~[Hexo 中添加 emoji 表情](http://www.cnblogs.com/fsong/p/5929773.html)~~ [Hexo 中添加 emoji 表情](http://chaxiaoniu.oschina.io/2017/07/10/HexoAddEmoji/)
* [为 Hexo 添加可折叠的文章目录](http://moxfive.xyz/2016/06/13/hexo-collapsible-toc/)
* [hexo-renderer-markdown-it 在 github 上的文档](https://github.com/celsomiranda/hexo-renderer-markdown-it/wiki/Advanced-Configuration)
* [hexo-renderer-markdown-it 在 npm 上的文档](https://www.npmjs.com/package/hexo-renderer-markdown-it)


