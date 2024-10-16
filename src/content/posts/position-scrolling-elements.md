---
title: 一个小技巧，快速定位滚动元素
date: 2024-08-23
summary: 工作中碰到需要手动触发元素滚动到指定位置，但由于嵌套的div太多，试了几个dom都不对，用下面的方式解决了
category: 技术分享
tags: [Develop, Front]
---

## 工作中碰到需要手动触发元素滚动到指定位置，但由于嵌套的div太多，试了几个dom都不对，用下面的方式解决了。上代码：

```
const isScrollable = (el: any) => {
   return el.scrollWidth > el.clientWidth;
};
window.addEventListener(
 'scroll',
  function (e) {
    let el = e.target;
    while (el && el !== document) {
    if (isScrollable(el)) {
       console.log('当前滚动的元素是:', el);
       break;
    }
  }
}，true);
```

如果是要查看纵向滚动的元素，scrollWidth改为scrollHeight即可；

另外我想说，GPT大法好哈哈😄
