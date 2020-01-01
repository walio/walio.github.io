---
title: mysql踩坑
date: 2020-01-01 22:27:52
updated: 2020-01-01 22:27:52
tags:
---
mysql踩坑系列
<!--more-->
（4）使用java或者python查询sql时，sum函数返回值映射的类型均为decimal，真是坑爹，参考https://stackoverflow.com/questions/10592481/what-is-the-return-type-of-sum-in-mysql，



没法在mysql里面验证，像insert into table1 select sum(column) from table2都会自动转换类型，可以使用cast(sum(column) as unsigned)将结果强转为int类型。

顺带一提

（5）联系第四条，用round是无法将decimal转换为int的，round函数仅会四舍五入而不会改变值类型



floor函数同理



（6）mysql存储非大文本类型的变量一行最长65535，对于utf8类型varchar来说，一个字节占3位，并且有一个字节存储varchar的长度，所以单个varchar最长为65535/3-1
