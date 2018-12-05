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
## hive
#### 优化
1. 调整map和reduce的大小
两个参数：最大值和分片大小
2. 数据倾斜
set hive.groupby.skewindata = true
针对count(distinct)，限制条件https://blog.csdn.net/baidu_29843359/article/details/46967473，具体原理：两轮map：
set hive.auto.convert.join = true;
针对小表join大表，将小表加载至内存，在大表的map端直接过滤，注意顺序不能乱，否则无法达到性能的提高（虽然不会报错）
#### hive sql
select 1/0不会报错，只会返回NULL，日了狗了
很多时候数据有问题真的不是逻辑问题啥的，多半是你自己犯了一个愚蠢到无以复加的错误
hive中的变量和字符串一样，比如表达式exp的最终结果是123，则arr[exp]其实是arr['123']，需要arr[cast(exp as int)]
关键字如current是不能作为如with as的标识符的，否则会报错
## 关系型数据库
#### sql cache
会通过sql语句生成cache来缓存，这也是为什么要确立大小写规范的原因




## hive.groupby.skewindata
一直不太理解hive.groupby.skewindata这个参数到底是如何解决数据倾斜的，很多时候我感觉即使加了效果也不大

## 数据倾斜
追根溯源到底distinct和count distinct为什么会引起数据倾斜？到底又是怎么解决的？
