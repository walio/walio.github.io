---
title: spark每日进步
tags:
  - scala
keywords:
  - spark
  - spark调试
  - spark广播
categories:
  - 数仓开发
---
写wiki感觉不太好，直接写个人博客。其中掺杂了太多个人理解，不保证正确性。但是感觉网上的都是官样文章，而且都是抄来抄去，真真叫没意思。
<!--more-->
1. 新手村
推荐一个[极好的博客](https://github.com/JerryLead/SparkInternals)，最好是一边看能够一遍动手验证，看看scala的函数是怎样转化成不同的stage和task的，看看spark-sql是怎么划分的，理解比较深入
2. 相关原理
* job划分和stage划分
凡是有action的地方会划分为一个job，个人理解action就是一种”落地“，之前做的所有计算都只是生成dag，都是空中楼阁，没有真正进行计算，而action会真正得到一个结果，真正的scala的数据结构array，可以进行普通的非spark的计算
宽依赖和窄依赖
凡是遇到shuffleDependency都会划分为两个stage，shuffleDependency大白话讲就是要计算下一个rdd的任何一个partition，都必须把上一个rdd的所有partition都计算出来，这个时候把两个rdd划分在同一个stage是不合适的，因为必须先把上一个rdd完全计算出来以后才能进行下一步的计算，所以就不必计较分区了。
所谓划分在一个stage，就是说下一个rdd的分区永远只依赖上一个rdd的部分分区，所以一直往前推，最后一个rdd的分区都只依赖这一个stage最早的rdd的部分分区，所以可以按照这个依赖关系，最后一个rdd的每个分区都划分为一个任务（task），这个分区的所有计算都在这个task内完成，依赖从后往前推算，而计算的流程是从前往后，后一个rdd的分区计算完成以后，前面的依赖分区就可以扔掉了（当然某些情况下比如笛卡尔积应该还是需要缓存一下？）。这样的话，就不需要每次计算完一个rdd就全部存储起来（无论是内存还是硬盘），而是根据数据流，计算完下一个rdd的某个分区，它依赖的上一个rdd的分区就可以不用了，”按需计算“
* driver和executor
可以认为driver只有一个，但是executor有多个，driver处理rdd等，executor是具体执行的单位，所以是无法在map的函数中执行spark.sql的，因为执行map中函数的操作是在executor上进行的，而‘解析’rdd只能在driver上做
* cache机制

3. 具体编程
* spark调试
一开始以为print操作只会在各个executor上执行，所以并没有打印出来，其实只是因为map是transformation操作，没有调用action时是不会执行的，调用collect方法以后会发现rdd中的print操作都执行了。
* rdd.map和rdd.foreach的区别：
https://stackoverflow.com/questions/41388597/difference-between-rdd-foreach-and-rdd-map
（1）map有返回值，foreach不会返回任何值（调用过foreach后rdd会变为None）
（2）map操作不会立即执行，但foreach操作会
（3）理论上来说foreach适用于某种操作，如调用restful api等，map适用于对数据做映射，
* spark的广播
![“这张图讲的很好”](/img/broadcast.png)虽然rdd和map函数都定义在同一个python文件中，但是实际上map函数是在不同的excutor上执行的，可以通过闭包的方式将变量传递给各个rdd（在py文件顶层定义变量，在map的函数中使用），但这种方式，每次executor遇到不属于rdd的变量时都会请求一次driver，而广播变量则是一开始申请一次并缓存起来，后面每次都使用这个缓存。
