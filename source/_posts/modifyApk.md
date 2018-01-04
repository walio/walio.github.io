---
title: 随便讲一讲修改APK重新签名打包
date:
tags:
  - 安全
keywords:
  - apk修改
  - apk注入
categories:
---
最近做的apk检测，随便讲一讲修改apk重新打包以获取输出方便调试的方法。[参考](吾爱破解的链接)
<!-- more -->
## 起因
在做apk检测的时候，已经发现了app通信的数据包是明文（https协议但未校验证书），但窝火的是数据包没办法修改，有一个sign校验值
![“数据包”](/img/modifyApk-1.png)
修改了内容就会说校验值不通过
![“数据包”](/img/modifyApk-2.png)
当下就起了歹心，这明显就一个md5加密字符串嘛，大致上肯定也就是请求包里各个字段拼接起来在加密，加上这个app甚至各个历史版本都可以在更新的服务器下载，想说这也太不安全了，一定有问题，怎么能被一个区区的sign值困住？于是开始了反编译之路。反编译自然不说了，apktool、Android Killer之类的，搜索到sign字符串，大致定位为其中一个文件。
![“sign加密”](/img/modifyApk-4.png)
一开始把log日志打印出来看，不过到底还是开发人员的调试日志，没有自己想要的结果，不如自己加的爽
参考吾爱破解的文章，自己做了一个[crack类](https://gist.github.com/walio/ca674e4a7838fc9b3026134a81e01fb4)，调用toString方法打印对象到/sdcard/debug.txt，毕竟像smali源码中StringBuilder这样的对象就不能通过多态，以打印String的方法打印；另外数组需要特别地使用Array.toString方法，如果直接调用toString方法，打印出来会是像[B@c618d72这样的乱七八糟的东西（“[”表示是数组，“B”表示Byte，“@c618d72”表示地址）
## 使用方法
在apktool反编译出的smali、smali_classes2这些随便一个文件夹下直接放入crack.smali，调用的时候按照crack.smali的main方法里那样调用。
需要注意的是如果需要添加tag，在函数的寄存器个数声明处，一般是在函数开头的第一行，给寄存器个数加一，比如将`.registers 5`改成`.registers 6`，毕竟tag字符串需要一个寄存器存储。
实际上我想这个技术应该不算新鲜，毕竟众厂商早就开发出了在apk中植入广告的技术，而且现在大家都提高了安全意识，想必重打包、重签名都畅通无阻的App也应该不多，不过对于一个新上手的人来说，这样玩还算有趣，有时间还可以研究研究内购破解什么的。
