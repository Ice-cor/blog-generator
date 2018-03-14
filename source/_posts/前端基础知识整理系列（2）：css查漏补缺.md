---
title: 前端基础知识整理系列（2）：css查漏补缺
date: 2018-03-08 02:20:36
tags: css
---
很多人说css太烦了，工作中使用的时候发现确实如此。要掌握好css，我觉得没有其他什么特殊的技巧，就是一个经验的问题。一个前端老手跟一个新手在css上的差距，就是谁用的次数多，熟能生巧嘛。当然，这里所说的是纯css，sass，less这些预处理语言先不说。我的个人经验就是不断的尝试，试错多了积累下来的就是你的经验。

## CSS 学习资源

学习css不建议看书，最好的书就是文档跟搜索引擎。

  1、 Google: 关键词 MDN
  2、 [CSS Tricks](https://css-tricks.com/)  // css效果网站
  3、 [Google: 阮一峰 css](https://www.google.com/search?q=%E9%98%AE%E4%B8%80%E5%B3%B0+css)
  4、 [张鑫旭的 240 多篇 CSS 博客](http://www.zhangxinxu.com/wordpress/category/css/page/25/)
  5、 [Codrops 炫酷 CSS 效果](https://tympanus.net/codrops/category/playground/)  //css+js
  6、 CSS揭秘  //书籍
  7、 [CSS 2.1 中文 spec](http://cndevdocs.com/)
  8、 [Magic of CSS 免费在线书](http://adamschwartz.co/magic-of-css/)

*建议用谷歌浏览器，国内的浏览器可能会搜出很多奇奇怪怪的东西。。。其次一定要会翻墙。。*

## 一些总结

1、transition的用法

```
.a{
    transition:  box-shadow 1s;
}
.a:hover{
    box-shadow: 10px 10px 10px black;
}
//可以延缓hover的效果，使其有动画。
```

2、加了display: inline-block的元素，会在下方出现空隙，解决办法是加上vertical-align: top

3、例如img这些行内元素可能会出现1px的bug，可以对其加上display：block解决

4、对不确定宽高元素的100%居中

```
.a{
    position: absolute;
    top: 50%;
    left: 50%;
    transform: translate(-50%,-50%);
}
```

5、对html、body设置{width: 100vw; height: 100vh; }比} width: 100%; height: 100%;}好用，因为不用根据父级元素来判定宽高，而是根基视图区域

6、**想到再更。。。**

*善用搜索引擎*