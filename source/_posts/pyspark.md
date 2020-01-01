---
title: pyspark相关问题踩坑
date: 2020-01-01 21:57:30
updated: 2020-01-01 21:57:30
tags:
---
多年以来踩过许许多多spark的坑，查问题的时候也是各种艰辛，基本没有中文资料，记录一下防止自己忘记，也方便后来人，主要是分pyspark相关、sparksql相关。按照问题与回答进行组织。本篇为pyspark相关问题。
<!--more-->
*pyspark本身没有什么问题，主要是pyspark中用到numpy等会遇到各种各样的问题。*
* **numpy的dtype转换**
<!-- -->
astype，直接int可以转为python基本类型
* **rdd join时严格按照key-value**
<!-- -->
rdd left join时，比如rdd是(a,b,c,d,e,f) left join任何rdd，则后面的cdef均会丢失
* **spark的udaf相关**
<!-- -->
spark dataframe 自定义udaf需要借助pandas的帮忙，比较麻烦（有没有别的方法？）
* **pyspark序列化异常**
<!-- -->
```
Caused by: net.razorvine.pickle.PickleException: problem construction object: java.lang.reflect.InvocationTargetException
at net.razorvine.pickle.objects.AnyClassConstructor.construct(AnyClassConstructor.java:29)
at net.razorvine.pickle.Unpickler.load_reduce(Unpickler.java:707)
at net.razorvine.pickle.Unpickler.dispatch(Unpickler.java:175)
at net.razorvine.pickle.Unpickler.load(Unpickler.java:99)
at net.razorvine.pickle.Unpickler.loads(Unpickler.java:112)
at org.apache.spark.api.python.SerDeUtil$$anonfun$pythonToJava$1$$anonfun$apply$1.apply(SerDeUtil.scala:188)
at org.apache.spark.api.python.SerDeUtil$$anonfun$pythonToJava$1$$anonfun$apply$1.apply(SerDeUtil.scala:187)
at scala.collection.Iterator$$anon$12.nextCur(Iterator.scala:434)
at scala.collection.Iterator$$anon$12.hasNext(Iterator.scala:440)
at scala.collection.Iterator$$anon$11.hasNext(Iterator.scala:408)
at scala.collection.Iterator$$anon$11.hasNext(Iterator.scala:408)
at scala.collection.Iterator$$anon$11.hasNext(Iterator.scala:408)
at org.apache.spark.sql.catalyst.expressions.GeneratedClass$GeneratedIteratorForCodegenStage1.agg_doAggregateWithoutKey_0$(Unknown Source)
at org.apache.spark.sql.catalyst.expressions.GeneratedClass$GeneratedIteratorForCodegenStage1.processNext(Unknown Source)
at org.apache.spark.sql.execution.BufferedRowIterator.hasNext(BufferedRowIterator.java:43)
at org.apache.spark.sql.execution.WholeStageCodegenExec$$anonfun$10$$anon$1.hasNext(WholeStageCodegenExec.scala:614)
at scala.collection.Iterator$$anon$11.hasNext(Iterator.scala:408)
at org.apache.spark.shuffle.sort.BypassMergeSortShuffleWriter.write(BypassMergeSortShuffleWriter.java:131)
at org.apache.spark.scheduler.ShuffleMapTask.runTask(ShuffleMapTask.scala:96)
at org.apache.spark.scheduler.ShuffleMapTask.runTask(ShuffleMapTask.scala:53)
at org.apache.spark.scheduler.Task.run(Task.scala:109)
at org.apache.spark.executor.Executor$TaskRunner.run(Executor.scala:345)
at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1142)
at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:617)
... 1 more
```
类似于如上所示PickleException的问题，是因为在spark的action操作时返回的不是基本数据类型，没有实现writable接口，java无法进行序列化从而在各个节点间传输数据导致的。可以检查下返回的数据中是否有numpy数据类型或nan等数据。
* **nan问题**
<!-- -->
numpy在类似于0/0问题上不会报错，可以通过设置np.seterr(all='raise')让其抛出异常。但是seterr方法对np.mean函数无效，np.mean([1,2,nan])仍然可以正常运行。Decimal(np.nan)和Decimal.from_float(np.nan)均能正常运行得到一个神奇的Decimal('NaN')并且不会报错，设置seterr同样无用。

* **rdd中数据类型不一致**
<!-- -->
当调用spark.sql().rdd某一列为Decimal类型，并向其中插入部分int类型数据时，spark会尝试使用decimal取解析，从而将int解析为None。（赋值给np矩阵就会变为nan）
