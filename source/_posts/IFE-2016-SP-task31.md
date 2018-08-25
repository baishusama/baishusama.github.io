---
title: IFE-2016-SP-task31 Linkage Inputs
date: 2017-01-27 15:25:32
tags: [IFE]
---

## My Solution

My solution to task31 is available on jsfiddle: 
{% jsfiddle 45v3nuLc 'default' 'light' '100%' '400px' %}

<!-- more -->

### 利用“复选框 hack”来实现类似标签页的切换效果

`input[type="radio"]:checked ~ section.class` 来切换 `section` 的显示/隐藏。

```css
/* 下面的 #student 和 #notStudent 是两个 input[type="radio"] */
/* 下面的 .s1 和 .s2 是两个 section */
#student:checked ~ .s1 {
  display: block;
}

#notStudent:checked ~ .s2 {
  display: block;
}

section {
  display: none;
}
```

> 《CSS 揭秘》P152

## Compared with others'

### [任务得分第一的团队的 solution](http://ife.baidu.com/2016/review/detail?workId=4198)

* 获取 `select` 当前值（见本文最后）

### [任务得分第二的团队的 solution](http://ife.baidu.com/2016/review/detail?workId=4198) 和 [任务得分第三的团队的 solution](http://ife.baidu.com/2016/review/detail?workId=4198)

* :grey_question: （“城市”“学校”相关）数据存储在？：数组 VS 对象
    - [存储在数组](https://github.com/Sakitama/Sakitama.github.io/blob/master/js/task-31.js)：
        + 顺序有保证
        + 结构不是那么清晰
    - [存储在对象](https://jsfiddle.net/baishusama/45v3nuLc/)：
        + `for-in` 可能顺序不理想
        + 结构更清晰（城市学校的对应关系明了）
    - [存储在数组和对象的嵌套结构中](https://github.com/SublimeUs/sublimeus.github.io/blob/master/task31/frank/js/task.js)：
* :question: 事件代理的兼容性处理（必要？？？）
    ```javascript
    function delegateEvent(elem, tag, event, listener) {
      addEventHandler(elem, event, function () {
        var e = arguments[0] || window.event,
            target = e.target || e.srcElement;
        if (target && target.tagName === tag.toUpperCase()) {
          listener.call(target, e);
        }
      });
    }
    ```

## 一些知识点

### :bookmark: 获取 `select` 当前值

> [Get Value or Selected Option in Select Box](http://www.dyn-web.com/tutorials/forms/select/selected.php)
> [Get selected value in dropdown list using JavaScript? @SO](http://stackoverflow.com/questions/1085801/get-selected-value-in-dropdown-list-using-javascript)

#### 多种方式

> P.S. 仅适用于 `sel.type === "select-one"` 的 `select`；不适用于 `sel.type === "select-multiple"` 的 `select`。同时适用于两种 `type` 的 `select` 值的获取方式可以参考最后 jQuery 的代码片段。

* 方式一：`.value`
    - 存在兼容性问题：IE9 以下在 `option` 没有 `value` 值的情况下，无法像别的浏览器那样返回 `option` 的文本内容。
* ~~方式二：`selectedOptions`（这个方法是我自己试出来的。。好像提到的不多，可能不可靠？？）~~
    ```javascript
    sel.selectedOptions[0].value
    sel.selectedOptions[0].text
    ```
* 方式三：`selectedIndex`（:bookmark: 常用方法）
    ```javascript
    sel.options[sel.selectedIndex].value
    sel.options[sel.selectedIndex].text
    ```
* 方式四：`for` 循环判断 `option` 的 `selected` 值

#### 扩展阅读：jQuery 中 `select` 的 `.val()` 方法的实现

```javascript
select: {
  get: function( elem ) {
    var value, option,
        options = elem.options,
        index = elem.selectedIndex,
        one = elem.type === "select-one",
        values = one ? null : [],
        max = one ? index + 1 : options.length,
        i = index < 0 ?
            max :
            one ? index : 0;

    // Loop through all the selected options
    for ( ; i < max; i++ ) {
      option = options[ i ];

      // Support: IE <=9 only
      // IE8-9 doesn't update selected after form reset (#2551)
      if ( ( option.selected || i === index ) &&

            // Don't return options that are disabled or in a disabled optgroup
            !option.disabled &&
            ( !option.parentNode.disabled ||
              !jQuery.nodeName( option.parentNode, "optgroup" ) ) ) {

        // Get the specific value for the option
        value = jQuery( option ).val();

        // We don't need an array for one selects
        if ( one ) {
          return value;
        }

        // Multi-Selects return an array
        values.push( value );
      }
    }

    return values;
  },

  set: function( elem, value ) {
    // Omitted owing to the limitation of space
  }
}
```


