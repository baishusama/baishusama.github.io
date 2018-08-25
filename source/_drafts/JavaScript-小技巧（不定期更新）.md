---
title: JavaScript 小技巧（不定期更新）
tags: [javascript]
---

### 获取当前时间的毫秒数

```javascript
// Way 1. 直接的方式
var time = new Date().getTime();

// Way 2. 利用隐式类型转换
var time = +new Date();
```