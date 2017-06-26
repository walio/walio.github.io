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

## 基本语法
#### 参数列表
需要指定非字面量的默认参数时，表达式只会执行一次。
```python
def log(message, time=datetime.now()):
  print_log()
```
这样的话datetime.now()只会在函数定义时执行一次，以后每次调用函数time都一样。同样
```python
def decode(data, default={}):
  decode_data()
```
{}只会执行一次，所有的函数调用共享同一个字典。解决方法是，将默认参数赋值为None，在docstring中说明其作用，在函数体中对其赋值。
#### 作用域
python只有def才会分割作用域。
1. 内部均可访问外部。
2. if、for不分割作用域，外部可访问内部；def分割，外部不能访问内部。
3. nonlocal会去外部找变量，一直找到全局(此py文件)的“第一层”def

#### 字典的get方法
dict.get方法和dict["attr"]的不同在于get方法访问不存在的属性时返回None，而dict["attr"]会报KeyError的异常。谈不上哪个好，应用场景不同，有时候需要捕捉异常，有时候不需要。
```help
get(...)
    D.get(k[,d]) -> D[k] if k in D, else d.  d defaults to None.
```
#### 内置函数
1. filter,map,reduce,zip在python3中均变为迭代器了，不过使用for遍历的方法依然一样。
2. sum函数默认初始的数为0，所以` sum([[1,2],[3,4]]) `会报` TypeError: unsupported operand type(s) for +: 'int' and 'list' `的异常，就是因为会尝试0+[1,2]的操作，正确的方法应该是指定初始值` sum([[1,2],[3,4]],[]) `
```python
def sort_priority(values, group):
  def helper(x):
    if x in group:
      return (0, x)
    return (1, x)
  values.sort(key=helper)
```
   函数作用是对values排序，将values中在group的部分放到前面去，其余部分默认排序。原理是比较元祖时，会从第一位开始每一位进行比较。

#### 串行迭代器

```python
In [21]: a = [1,2,3,4]

In [22]: it1=(x*2 for x in a)

In [23]: it2=(x**2 for x in it1)

In [24]: next(it1)
Out[24]: 2

In [25]: next(it2)
Out[25]: 16

In [26]: next(it1)
Out[26]: 6

In [27]: next(it2)
Out[27]: 64
```
外部迭代器的next()会推动内部next()一次。
本来以为使列表通过多个过滤器只有` filter(lambda: fil1(fil2()), somelist)`，忽然发现可以` filter1 = filter(fil1, somelist);filter2 = filter(fil2, filter2)`
```ipython
In [28]: def fil1(x):
    ...:     return x>0
    ...:

In [29]: def fil2(x):
    ...:     return x>2
    ...:

In [30]: a=[-1,1,3,1,3,-1]

In [31]: filter1 = filter(fil1,a)

In [32]: filter2 = filter(fil2,filter1)

In [33]: next(filter1)
Out[33]: 1

In [34]: next(filter2)
Out[34]: 3

In [35]: next(filter1)
Out[35]: 1

In [36]: next(filter2)
Out[36]: 3
```
#### 同时遍历多个列表
用zip(list1, list2)保持下标一致，短列表遍历完后结束。若想取消提前终止，使用itertools中的zip_longest
#### try/except/else
这里的else是try成功执行才会执行，和直接写不一样，直接写是无论成功失败都会执行。
finally则是无论成功失败都会执行，最大用途是打开文件时，向上传递异常之前执行关闭文件操作。
#### __call__()和__init()
定义了__call__方法的实例（不是对象）可以当做函数看待。
```ipython
class test:
    def __init__(self):
        print("init")
    def __call__(self):
        print("call")


test()
init
Out[6]: <__main__.test at 0x1f6d2e0d9b0>

test()()
init
call
```
#### tricks
list()对字典操作时只会讲键编为列表
