---
title: sparksql踩坑实践
date: 2020-01-01 21:36:45
updated: 2020-01-01 21:36:45
tags:
---
多年以来踩过许许多多spark的坑，查问题的时候也是各种艰辛，基本没有中文资料，记录一下防止自己忘记，也方便后来人，主要是分pyspark相关、sparksql相关。尽量按照问题与回答进行组织。本篇为sparksql相关问题。
<!--more-->
* **为什么与decimal相乘时，hive和sparksql有精度差异？**
<!-- -->
`select (cast(29.26831421 as decimal(20,8))) * cast(4532 as bigint)`，乘法真实结果是132643.99999972，在hive中结果是132643，sparksql结果是132644。乍一看是spark做了四舍五入hive去尾，实际不是。
首先cast as int无论spark还是hive都是去尾。通过explain分析等可以得出，hive执行时，select cast(29.26831421 as decimal(20,8)) * cast(4532 as bigint)并不会丢失decimal精度，而sparksql在这一步已经变为132644，具体原因是spark首先将bigint转换为decimal(20,0)（猜测因为bigint是8byte，范围-9,223,372,036,854,775,808 ~ 9,223,372,036,854,775,807，整数19位+符号1位），再取上限两个数均转换为decimal(28,8)，然后结果保留6位小数（详见2），实际上是132643.99999972舍入为132643.000000

* **关于保留小位数**
<!-- -->
decimal\*any，会将any转换成decimal，如bigint转为decimal(20,0)，int转为decimal(10,0)，然后成为decimal(a,b)\*decimal(c,d)的计算，计算结果的精确度首先保证整数部分，保留位数为38-(a-b)-(c-d)，如decimal(20,8)\*bigint保留位数为38-12-20=6

* **为什么collect_list的数值类型会被转为string类型？**
<!-- -->
collect_list只能collect同一类型数据，当spark.sql中有collect_list(date,value)之类的语句时，value会被强转为string

* **多个collect_list时，list中不同列的相对位置是否相同？**
<!-- -->
貌似并无保证。[refer1](https://stackoverflow.com/questions/44448008/does-collect-list-maintain-relative-ordering-of-rows)，[refer2](https://stackoverflow.com/questions/40407514/use-more-than-one-collect-list-in-one-query-in-spark-sql)

* **为什么insert会生成重复记录？**
<!-- -->
insert overwrite会覆盖，insert不会，会生成重复记录，注意粘贴别粘错

* **为什么array(key, value)的时候将value转为Boolean时后一位永远是true？**
<!-- -->
原理就是array中只能存同样类型的值，`select array(a,b)`，a为string而b为int时，会自动执行`select(a,cast(b as string))`，取出后都为字符串，小心使用1和0表示true和false的，因为'1'和'0'均为true

* **关于int的上限**
<!-- -->
select cast(xxx as int)注意数值范围，超过int范围会变为负值，建议统一用cast(xxx as bigint)

* **下述语句会生成731行记录**
<!-- -->
```
SELECT
  DATE_ADD('{p_date}', index - 365) AS base_date
lateral view posexplode(split(space(730), ' ')) pe as index,s
```

* **with as会进行缓存吗？**
<!-- -->
with as貌似并不会缓存，如下面的查询中多次使用时的filter和select的列均相同时，会复用，其他情况则不会

* **window语句为什么不能用在where条件里面？**
<!-- -->
https://stackoverflow.com/questions/13997177/why-no-windowed-functions-in-where-clauses

<!-- todo -->
* **hive表添加列以后为什么新增列都是空？**
<!-- -->
hive更改列名后列值会变为null，可以使用`ALTER TABLE table SET TBLPROPERTIES ('parquet.column.index.access'='true')`使用列的次序而不是列名读取列，可见aws[官方文档](https://docs.aws.amazon.com/zh_cn/athena/latest/ug/handling-schema-updates-chapter.html)

* **percent_rank重复记录的percent是一样的**
<!-- -->
* **为什么hash()方法冲突很严重？**
<!-- -->
sparksql的hash()取值空间为int上下限（存疑），sparksql使用的是murmur3hash（确定），murmur3hash有2个版本：https://segmentfault.com/a/1190000010990136
> 除了 128 位版本以外，它还有生成 32 位哈希值的版本。在某些场景下，比如哈希的对象长度小于 128 位，或者存储空间要求占用小，或者需要把字符串转换成一个整数，这一特性就能帮上忙。当然，32 位哈希值发生碰撞的可能性就比 128 位的要高得多。当数据量达到十万时，就很有可能发生碰撞。
<!-- -->
猜测sparksql使用的是32位版本，哈希冲突在sqrt(N)的时候大概率发生冲突，目前来看10万的数据量就会大概率发生冲突。int加密到int参考解决方案：conv(substring(md5(string(did % (some prime))), start_index), 16, 10)。
2019-12-11更新：确认sparksql用的hash是32位版本的，故在2^16量级发生碰撞概率较大。理论上来书spark的哈希适用于非加密转换

* **sparksql为什么插入的时候特别慢？**
<!-- -->
sparksql insert overwrite太慢是因为使用了hive的早期版本，其中删除文件用了for，而hive后续版本使用了线程池 https://www.cnblogs.com/barneywill/p/10154922.html

* **为什么取最新分区数据时速度很慢？**
<!-- -->
首先，以下三者是相同的
`select * from table where p_date in (select max(p_date) from table)`
`select * from table inner join (select max(partition_col) max_p from table t1) a where table.patition_col=a.max_p`
`select * from table left semi join (select max(partition_col) max_p from table t1) a where table.patition_col=a.max_p`
explain一下就能明白，三者的plan是一样的，改一种写法根本没有用（很容易理解，这些写法是完全等价的，spark有optimize，如果其中一种方法更快的话spark一定会把另外一个写法优化为更快的那种）测试版本：spark2.3。
但是两种都很慢，因为这样会扫描全表。为什么扫描全表？（感觉很有问题，为什么不是扫描meta信息后取出符合条件的分区扫描）参考[这个问题](https://stackoverflow.com/questions/56994923/spark-subquery-scan-whole-partition) 题主还特地在评论中说问为什么，但没有人回答为什么，全是应该怎么办，也是无语了。个人正在等待[回答](https://stackoverflow.com/questions/59375233/sparksqlwhy-in-clause-on-partition-col-lead-to-full-table-scan)中 （todo：完善一下问题）

* **hive如何实现乘法聚合？**
<!-- -->
hive竟然没有类似sum的乘法聚合，坑爹！类似exp(sum(log(value)))勉强能实现，sf上全是这种，但是其实是错的！负数我且不论了，遇到0都很尴尬，value中有一个0，则乘法聚合结果理论上来说应该是0，然而这种算法，log(0)=null，因此聚合时会忽略这个null！而且还很难排查，因为不会报错，结果也很正常，不是null，[这篇文章](https://blog.jooq.org/2018/09/21/how-to-write-a-multiplication-aggregate-function-in-sql/)做了非常深入的探讨,搭嘎，口头哇路！这么麻烦不难受么，直接udf多好

* **为什么float类型每次加和都不同？**
<!-- -->
float类型加和有精度问题，由于不同的executor执行速度不一样，每一次可能是不同的executor的和聚合加在一起，因此进行舍入时的值时不一样的，会造成每次加的时候都会有一点点非常小的差异。

* **多个limit语句union需加括号**
<!-- -->
select语句中有limit语句时必须加括号，也就是`select * from t1 limit 10 union all select * from t2 limit 10会报错，必须改为(select * from t1 limit 10) union all (select * from t2 limit 10)`，[参考](https://stackoverflow.com/questions/1415328/combining-union-and-limit-operations-in-mysql-query)，原因是需要指定limit语句生效的子句

* **union all只会检查字段数是否一致而不会检查字段名**
<!-- -->
`select a,b from table1 union all select c,d from table2`是正确的，生成的字段名分别为a和b。所以字段顺序很重要，`select a,b from (select a,b from table1 union all select b,a from table2)`也是完全正确的，看不出来问题，真正用的时候才会有问题。另外insert select也务必注意顺序，若两列类型相同但顺序错了也是能插入的。
