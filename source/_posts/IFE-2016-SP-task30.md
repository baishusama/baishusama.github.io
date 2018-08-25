---
title: IFE-2016-SP-task30 Verify Multiple Inputs
date: 2017-01-26 22:08:56
tags: [IFE]
---

## My Solution

My solution to task30 is available on jsfiddle: 
{% jsfiddle q989o3nm 'default' 'light' '100%' '400px' %}

<!-- more -->

* :confused: 这个实现中不太优雅的地方：
    - 分别为每个 `input` 都定义了一个 `check 函数`，虽然有利用 `checkValue` 复用一部分代码。。
    - 为了改变 `this` 使用了很多很多的 `.call()` 方法。。
    - `blur` 的事件委托（历史遗留问题）。。
* :confused: 这个实现中不太可靠的地方：
    - 各项的验证规则（正则）
    - 寄生组合式继承
      + [ ] 直接给 `Error` 构造函数传递 `message` 参数无效。。（See Line 91）

## Compared with others'

### [任务得分第一的团队的 solution](http://ife.baidu.com/2016/review/detail?workId=8817)

* :ribbon: 巧妙地各种运用数组及其方法
    - `hintText` 数组
        ```javascript
        var hintText = [{ hint: "必填，长度为4~16位字符", right: "名称格式正确", wrong: "名称格式有误", isPassed: false },
          { hint: "必填，长度为4~16位字符,包含字母和数字", right: "密码可用", wrong: "密码不可用", isPassed: false },
          { hint: "必填，必须与密码相同", right: "密码输入一致", wrong: "密码输入不一致", isPassed: false },
          { hint: "填写正确的邮箱格式", right: "邮箱格式正确", wrong: "邮箱格式错误", isPassed: false },
          { hint: "必填，长度为4~16位字符", right: "手机格式正确", wrong: "手机格式错误", isPassed: false }
        ];
        ```
    - 利用数组的 `.forEach()` 和 `.every()` 方法来验证所有输入
        ```javascript
        // 我的写法不优雅之处：
        // 1. 为每个 input 分别定义了检查函数，需要传入相关 dom 元素。
        // 2. 提交验证时，if 判断条件由一堆很长的 && 组成。
        addEventHandler(verifyBtn, "click", function() {
          var rawVal = nameDOM.value;
          if (checkName.call(nameDOM) && checkPwd.call(pwdDOM) && checkRepeatPwd.call(repeatPwdDOM) && checkEmail.call(emailDOM) && checkMobile.call(mobileDOM)) {
            alert("提交（验证)成功！");
          } else {
            alert("请按提示输入正确的信息后，再提交。");
          }
        });
        // 该团队的写法：
        // 1. 由于在 html 中定义了包含数字的 id 作为钩子，
        //    其检查函数只需传入对应的序号做参数即可找到相关 dom ，而不需要像我那样传入 dom 作为参数。
        // 2. hintText 在存储提示信息的同时存储了检查状态。
        //    在提交时，先是 .forEach() 地 checkValue 一遍，再利用 .every() 得到验证结果的 flag 值。
        addEventHandler(document.getElementById("submit"), "click", function(e) {
          e.preventDefault();
          [1, 2, 3, 4, 5].forEach(function(el) {
            checkValue(el);
          });
          var flag = hintText.every(function(singleHint) {
            return singleHint.isPassed;
          });
          if (flag)
            alert("提交成功");
          else
            alert("提交失败");
        });
        ```
    - **事件循环绑定** 可以利用数组的 `.forEach()` 方法 **避开闭包**（虽然这里真正避开闭包的原因是使用了节点的 id 中包含的数字，而没有使用 for 循环中的计数器 i ，但是使用`数组的循环方法`取代`通常的 for 循环`，确实是避开闭包的一种可行方案。）：
        ```javascript
        var inputs = document.getElementsByTagName("input");
        [].forEach.call(inputs, function(elem) {
          var id = elem.getAttribute("id").slice(1);
          var hintID = "h" + elem.getAttribute("id").slice(1);
          addEventHandler(elem, "focus", function() {
            document.getElementById(hintID).style.display = "table-row";
          });
          addEventHandler(elem, "blur", function() { checkValue(id) });
        });
        ```

### [任务得分第二的团队的 solution](http://ife.baidu.com/2016/review/detail?workId=8842)

* **事件循环绑定** 利用事件回调函数的 `e.target` 避开使用 for 循环的计数器 i （从而 **避开闭包**）

### [任务得分第三的团队的 solution](http://ife.baidu.com/2016/review/detail?workId=2691)

* [ ] `ES6` 待读。。

## 各项验证

### 手机号

* 我的 `/^\d{11}$/` ：11 位纯数字
* `/^1\d{10}$/` ：第一位为 1 的 11 位纯数字
* `/^1(3|4|5|7|8)\d{9}$/` ：考虑到前两位的 11 位纯数字
