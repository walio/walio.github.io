---
title: 常用的一些sql
date: 2020-01-01 22:02:01
updated: 2020-01-01 22:02:01
tags:
---
笔记系列，记录一些常用的sql
<!--more-->
1. 取多个列的最大值最小值：greatest,least
2. collect_map：str_to_map(concat_ws(",",collect_set(concat_ws(':', day, cast(value as string)))))
3. 类似python的range，生成一个空序列：posexplode(split(space(len)))
