---
title: sql优化相关
tags:
  - sql
keywords:
  - sql
categories:
---
sql看起来简单，还是有点东西的（不过还是很简单)
<!--more-->
## 踩坑

## 优化
1. 调整map和reduce的大小
两个参数：
2. 数据倾斜
set hive.groupby.skewindata = true
针对count(distinct)，限制条件https://blog.csdn.net/baidu_29843359/article/details/46967473，具体原理：两轮map：
set hive.auto.convert.join = true;
针对大表join小表，将小表加载至内存，在大表的map端直接过滤
