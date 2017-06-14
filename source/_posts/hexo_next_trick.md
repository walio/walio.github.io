---
title: next主题一些技巧
date:
tags:
  - hexo
keywords:
  - hexo
  - next
  - hexo生成的博客创建时间都相同
categories: 如何整一个没什么卵用的博客
---
一些小技巧，同样是没什么卵用的
<!-- more -->
## 文章排序杂乱
首先感谢[同学](https://github.com/ukari)发现并帮忙解决了这个问题。
hexo博客next主题是完全根据修改时间来给博文排序的，并没有提供类似于置顶等功能。详细讨论看[这里](https://github.com/iissnan/hexo-theme-next/issues/415)。可以通过[修改hexo-generator-index模块](http://www.netcan666.com/2015/11/22/%E8%A7%A3%E5%86%B3Hexo%E7%BD%AE%E9%A1%B6%E9%97%AE%E9%A2%98/)的方法魔改，但如果是通过travis-CI生成博客，node-modules一般都被gitignore，所以无效。个人还是等待next作者更新添加feature，我觉得希望还是比较大。
## 更新麻烦
