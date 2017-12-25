---
title: python每日进步
tags:
  - python
keywords:
  - python learning
  - urllib2和requests对比
categories:
---
## urllib2.urlopen和requests.get对比：
   1. 功能一样。urllib2.urlopen是异步调用，requests.get是同步调用，一旦有一个url阻塞住就无法继续往下了。grequests是异步的requests，但是无法取法取得响应信息。
   2. 自定义404界面requests.get可以正常返回页面信息，status_code属性是404等，而urllib2抛出HTTPError异常

## 打开文件
不要轻易用"w"打开文件，即使你什么也没做，文件也会被清空

## 性能测试
#### 工具
ipython，原谅我的愚昧无知，我一度以为ipython早就过时了，没想到功能很强大
#### 列表推导和map
老生常谈，[sf链接](https://stackoverflow.com/questions/1247486/python-list-comprehension-vs-map)，最好自己运行一遍，毕竟python更新之后性能也许会改变？Windows下如果` python -mtimeit -s'xs=range(10)' 'map(hex, xs)' `如果报`SyntaxError: EOL while scanning string literal`的错误的话，多半是因为单引号，改成
` python -mtimeit -s"xs=range(10)" "map(hex, xs)" `或者` python -mtimeit -s'xs=range(10)' 'map(hex,xs)' `，原因大概是`map(hex, xs)`中间的空格把这个字符串“截断成为两个参数`map(hex,`和` xs) `”了。
注意python3中map返回的是一个迭代器，返回列表需要list(map)。参考
[答案](https://stackoverflow.com/a/40948713/8131108)版本3.5.2和我版本3.6.1很接近就没有重复试验了
可以看出map相对于列表推导有着非常明显的优势，可能是python3对map做了特别的优化(妈蛋我怎么记得map是depricated建议用列表推导和zip替代？)
```ipython
%timeit list(map(sum, x_list))
10000 loops, best of 3: 161 µs per loop

%timeit [sum(x) for x in x_list]
1000 loops, best of 3: 212 µs per loop

%timeit list(map(sum, x_list))
10000 loops, best of 3: 161 µs per loop

%timeit [sum(x) for x in x_list]
1000 loops, best of 3: 213 µs per loop
```

```ipython
%timeit list(map(lambda i: i+1, i_list))
10000 loops, best of 3: 126 µs per loop

%timeit [i+1 for i in i_list]
10000 loops, best of 3: 53.9 µs per loop

%timeit list(map(lambda i: i+1, i_list))
10000 loops, best of 3: 125 µs per loop

%timeit [i+1 for i in i_list]
10000 loops, best of 3: 53.9 µs per loop
```
sf诚不我欺！
## 函数式编程
简单来说减少中间变量，尽量使用不可变的集合
[推荐博文](http://www.cnblogs.com/huxi/archive/2011/06/18/2084316.html)
#### 闭包
[简明介绍](https://zhuanlan.zhihu.com/p/22486908?refer=study-fe)简单来说就是防止随意更改变量，只允许部分函数更改
> 那么，如果想要利用reduce_找出列表中的最大值，应该怎么做呢？请自行思考：）

reduce_(lambda x,y:x if x>y else y,[1,2,3],1)
> 使用函数替代循环是函数式风格区别于指令式风格的最显而易见的特征。

利于并行？
###### 装饰器
