---
title: sql优化相关
date: 2018-10-12
updated: 2018-12-21
tags:
  - sql
keywords:
  - sql
categories:
  - data rd
---
sql看起来简单，还是有点东西的（不过还是很简单)，很多时候数据有问题真的不是逻辑问题啥的，多半是你自己犯了一个愚蠢到无以复加的错误
<!--more-->
以前觉得hive就是垃圾，太慢，sparksql支持的语法更全面而且更快，但实际上hive的稳定性远远好于sparksql，spark由于是内存计算，很容易崩，hive虽然慢但是能跑出来，但是只要hive跑不出来的东西，spark一定也跑不出来
## 语法
#### join on和join where的区别
left join中，on的条件除相当于where的加强版，where只会筛选出左=右的情况，而on相当于是where筛选出的记录+所有左记录
而inner join相当中on和where结果是一样的
#### union all select
sql中select的时候，列名是丝毫不重要的！只要对应位置的数据类型及总得列数相等就可以！神坑，select和union的时候很容易搞错，无论多久都会犯这个错误。
#### 0除的问题
select 1/0不会报错，只会返回NULL，日了狗了
#### 开窗函数：
grouping sets
sum(a) over(partition by b order)在hive下有的时候会有性能问题，hive跑了一个多小时根本没有进度，sparksql跑了两分钟（待查证）

hive中的变量和字符串一样，比如表达式exp的最终结果是123，则arr[exp]其实是arr['123']，需要arr[cast(exp as int)]
关键字如current是不能作为如with as的标识符的，否则会报错
## 优化
#### sql cache
会通过sql语句生成cache来缓存，这也是为什么要确立大小写规范的原因
#### mapjoin
针对小表join大表，将小表加载至内存，在大表的map端直接过滤，注意顺序不能乱，否则无法达到性能的提高（虽然不会报错）
最近发现with as很消耗硬盘io，不知道为啥，待查证，另外sparksql是否存在此问题？

#### 数据倾斜
一直不太理解hive.groupby.skewindata这个参数到底是如何解决数据倾斜的，很多时候我感觉即使加了效果也不大，需要追根溯源到底distinct和count distinct为什么会引起数据倾斜？到底又是怎么解决的？
解决方法：
1. 调整map和reduce的大小
两个参数：最大值和分片大小
2. 数据倾斜
set hive.groupby.skewindata = true
针对count(distinct)，限制条件https://blog.csdn.net/baidu_29843359/article/details/46967473，具体原理：两轮map：
set hive.auto.convert.join = true;

## sql解析
logical plan->优化->physical plan->
感觉是和编译的过程类似？先优化生成中间语言，然后通过后端生成目标语言？看来也可以分编译器前端和后端
