---
title: IFE-2016-SP-task29 Verify Input
date: 2017-01-11 19:44:24
tags: [IFE]
---

## My Solution

My solution to task29 is available on jsfiddle: 
{% jsfiddle 16nbyx5z 'default' 'light' '100%' '400px' %}

<!-- more -->

* 多层嵌套的 `if` 语句可以通过 `else if` 语句来简化
    - :confused: 我原来的代码：
        
        ```javascript
        if (val) {
          if (notContainWhitespace(val)) {
            if (isValidLength(val)) {} else {
              information = "用户名长度要在 4 ~ 16 个字符内！";
            }
          } else {
            information = "用户名不能包含空格！";
          }
        } else {
          information = "用户名不能为空！";
        }
        ```

    - :blush: 修改后的代码：

        ```javascript
        if (!val) {
          information = "用户名不能为空！";
        } else if (!notContainWhitespace(val)) {
          information = "用户名不能包含空格！";
        } else if (!isValidLength(val)) {
          information = "用户名长度要在 4 ~ 16 个字符内！";
        }
        ```

* 其他：姓名 vs 用户名
    
    这里我想当然地把原任务中要求的“姓名”改成了“用户名”，现在觉得有不妥 :confused: 。

    1. 首先，当然，两者的含义不一样。
    2. 更重要的是，姓名中间可以包含空格——比如，英文名至少是两部分，甚至更多。所以我的 solution 比别的团队的（按照要求来的）多了一个空格的判断，因为一般而言用户名中是不允许有空格的。

## Compared with others'

### [任务得分第一的团队的 solution](http://ife.baidu.com/2016/review/detail?workId=8390) 和 [任务得分第二的团队的 solution](http://ife.baidu.com/2016/review/detail?workId=4872)

* :bookmark: 利用字符串的 [`charCodeAt()`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/String/charCodeAt) 方法来计算字符串长度：

```javascript
function countLength(str) {
  var inputLength = 0;
  for (var i = 0; i < str.length; i++) {
    var countCode = str.charCodeAt(i);
    if (countCode >= 0 && countCode <=128) {
      inputLength += 1;
    } else {
      inputLength += 2;
    }
  }
  return inputLength;
}
```

* :question: 用两行三列的 `table` 来布局
    
    和该团队一样，这种写法我也见过。客观地说，是可以保持提示信息的宽度总是不超过正上方的输入框的。但是，由于过去 `table` 一度被滥用于全站布局，导致我对使用 `table` 来布局有些抵触 :no_mouth: 。关于这里用好还是不用好，这个我先保留意见 :pensive: 。

### [任务得分第三的团队的 solution](http://ife.baidu.com/2016/review/detail?workId=3421)

* 将提示信息和相应的类封装成了一个对象，并作为 UI 更新接口的参数

    比起我的相关逻辑而言，分工更加明确 ~~，而且入口单一（弱耦合）【后半句我不太确定，特别是“耦合性”，这里暂时先去掉】~~。

    - 第一部分的判断还算相似：
        + 我的代码：

            ```javascript
            if (!val) {
              information = "用户名不能为空！";
            } else if (!notContainWhitespace(val)) {
              information = "用户名不能包含空格！";
            } else if (!isValidLength(val)) {
              information = "用户名长度要在 4 ~ 16 个字符内！";
            }
            ```

        + 该团队的代码：

            ```javascript
            if(testStr.length === 0 ){
              paramObj = msgs.error_required;// 不能为空 “姓名不能为空”
            }else if( !lenReg.test(testStr)){
              paramObj = msgs.error_length;// 字符长度不对 “长度为4~16个字符”
            }else{
              paramObj = msgs.right;// 格式正确
            }
            ```

    - 第二部分，我用了一个 `if` 判断，而该团队得益于将提示信息视作一个对象（其实，真正避免了 `if` 语句的是其 `UIInterface` 方法里对 `className` 的粗暴操作，这点个人认为我的做法更稳妥 :flushed: ），而直接将其做为一个接口的参数：
        + 我的代码：

            ```javascript
            if (information) {
              invalidStylish(information);
            } else { validStylish(); }
            ```

        + 该团队的代码：

            ```javascript
            UIInterface(veBoxEle, paramObj); 
            ```

* :no_mouth: ~~`console.debug()` 这个在 MDN 上已经不推荐使用了~~  
* 一句话评价：能用正则的都用正则了 :joy:

## 正则匹配中文 

* :exclamation: 判断中文和 **编码** 有关
    
    > 做的时候因为犯懒没研究各种说法不一的本质原因，现在得知问题出在 **编码** 上。我做 task 的时候用的基本是 Sublime Text 3，而 Sublime 的默认编码是 `UTF-8` （详见 `Preferences->Settings Default->"default_encoding": "UTF-8"`）。
    > 
    > 关于字符编码的历史，[知乎上这个回答](https://www.zhihu.com/question/23374078/answer/69732605)描述得很生动易懂。

    所以，下面，我使用的正则对应的编码是 `GB2312` ；而考虑到 Sublime 的编码，我应该改为使用 `/[\u4E00-\u9FA5]/` 这样的 UTF-8 对应的正则。

* 我一开始直接匹配的双字节字符：`/[^x00-xff]/`：

    ```javascript
    var enReg = /[x00-xff]/g;
    var zhReg = /[^x00-xff]/g;
    var length = 0;
    length += value.match(enReg) && value.match(enReg).length;
    length += value.match(zhReg) && value.match(zhReg).length * 2;
    ```

* 上面这种写法可以改进为：

    ```javascript
    var zhReg = /[^x00-xff]/g;
    var length = zhReg.replace(zhReg, "aa").length;
    ```

    > 我在控制台测试的时候，`~` `-` `!` 等也被视为双字节字符了 OAO ！
    > 究其原因估计是遵循 `GB2312` 编码的时候，`~` `-` `!` 等在双字节字符中也存在对应编码。

* 前二团队将 `ASCII` 码以外的都：
    
    ```javascript
    // 在 for 循环内部
    var countCode = str.charCodeAt(i);
    if (countCode >= 0 && countCode <=128) {
      inputLength += 1;
    } else {
      inputLength += 2;
    }
    ```

* 第三团队匹配的是中文字符（应该不是全部，但基本涵盖常用？）：

    ```javascript
    var chineseReg = /[\u4E00-\uFA29]|[\uE7C7-\uE7F3]/g;
    var validLength = /^.{4,16}$/.test(trimmedStr.replace(chineseReg, '--'));
    ```
    
> **To be read**
> [ ] [What's the complete range for Chinese characters in Unicode? @SO](http://stackoverflow.com/questions/1366068/whats-the-complete-range-for-chinese-characters-in-unicode)
> [ ] [UTF-8 @wiki](https://www.wikiwand.com/zh/UTF-8)
> [ ] [GB2312 @wiki](https://www.wikiwand.com/zh/GB_2312)
