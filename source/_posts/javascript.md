---
title: 关于javascript
date: 2017-12-14
updated: 2017-12-14
tags:
  - JavaScript
  - 前端
keywords:
  - fe
categories:
  - fe
---
不知道是不是Python先入为主的原因，总感觉JavaScript是一门很扭曲的语言。比如for(let x in array)，x代表的是index，而且还是string；又比如a = {key:value}，key会被“自动解析”为字符串，所以(arg)=>{return {arg:"test"}}，函数体中的arg也是字符串"arg"，而非传入的参数arg；又比如我个人感觉，undefined的存在很鸡肋，不能说完全没有用，但是很容易让人混淆。很多语法都有种很反人类的感觉，又或者是我根本没有把JavaScript学懂？
<!-- more -->
## 关于原型链
#### 函数作为一般对象时
JavaScript中所有东西都是对象。大部分对象的含义都和传统面向对象一样， 单说下函数。查看函数的类型就是"function"。在作为对象这一点上，函数和和其他任何的对象没有任何区别。
```JavaScript
> function test(){}
< undefined
> typeof test
< "function"
```
每个对象都定义了一个__proto__属性，这个__proto__属性的值既是定义了对象的“类”，并且由于JavaScript中所有东西都是对象的思想，这个“类”也是一个对象，叫做原型对象。在这一点上函数和每一个对象一样。
#### 函数作为构造函数时
现在再来说说函数和其他对象不一样的地方。函数除了可以作为一个对象，还能作为一个类的构造函数。这个类定义在函数对象的prototype属性上。而且每个函数默认都有一个prototype是object（JavaScript默认对象）
```JavaScript
> function test(){}
< undefined
> test.prototype
< Object {constructor: function}
< constructor: function test()
< __proto__: Object
```
可以看到test.prototype仅有两个属性，constructor即是构造函数，__proto__是这个原型对象的原型。
#### 两种情况的异同
使用test()这种方式就是标准的函数调用，使用new test()就是将test当做构造函数新建对象。由于JavaScript中任何东西都是对象，所以任何东西都有__proto__属性来表述其原型对象（“类”），函数也是对象，函数的__proto__就是一个函数对象。
```JavaScript
> function test(){}
< undefined
> test.__proto__
< function () { [native code] }
> typeof test.__proto__
< "function"
```

但并不是每个对象都有prototype，只有函数有prototype属性。
另外函数作为一般函数调用时，this对象指的就是外层对象，在windows对象中定义的函数，this指向Windows。而作为类的构造函数时，this指向的是当前对象。

```JavaScript
> function test(){console.log(this)}
< undefined
> test()
< Window {stop: function, open: function, alert: function, confirm: function, prompt: function…}
< undefined
> new test()
< test {}
< test {}
```


#### 实例
原生的WebSocket比较难用，我需要扩展甚至改变原生对象的方法，比如连接上或断开以后设置某个标志位。当只有单个WebSocket时没问题，多个的话一个个写就太麻烦了。扩展的话没问题，直接WebSocket.prototype.newfunc=function(){}就行，然而改变是不行的，貌似无法改变原生对象的方法，考虑新建对象继承WebSocket。
1. mysock.prototype=WebSocket.prototype × mysock的原型就是WebSocket的原型，故无法改变
2. mysock.prototype=new WebSocket() × websocket需指定url参数
3. mysock.prototype=Object.create(WWebSocket) √


## sort方法
``` [-1,-2,0].sort() ```结果不对，非得``` [-1,-2,0].sort((a,b)=>{return a-b}) ```才对

## 遍历数组
for(var x in [1,2,3])，真难受，x是指数组下标，且是string
