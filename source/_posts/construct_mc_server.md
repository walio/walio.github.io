---
title: 搭建正义的太阳骑士Minecraft服务器(pc版和pe版)
date:
tags:
  - MineCraft
keywords:
  - Minecraft Server
  - 搭建mc服务器
  - 搭建mcpe服务器
categories:
  - 正义的太阳骑士套装
---
**pc版已经没问题，pe版调试中**
惊了，有[教程](https://www.linode.com/docs/game-servers/minecraft-on-debian-and-ubuntu)为什么我还要折腾半天
>请支持正版，本服务器仅做交流学习使用

| 客户端类型 | 客户端版本 | 客户端下载 | 服务器地址 | 客户端版本特性 |
| ------| ------ | ------ | ------ | ------ |
| Minecraft | v1.12 | [下载](http://pan.baidu.com/s/1boBkt7T)(x8tb) | 47.93.218.135 | [链你妹的接啊](http://minecraft-zh.gamepedia.com/1.12) |
| Minecraft: Pocket Edition | v | [下载]() | 47.93.218.135 | [再链一个试试]() |
<!-- more -->
pc版用的[官方版服务器](https://minecraft.net/zh-hans/download/server)，pe版用的[pmmp]()，大名鼎鼎的PocketMine-MP续作
说起来这个pmmp也是牛逼啊，好像是反编译mc然后用php重写了一遍？我等只有崇拜
## pe版建服
(非root用户去掉第三行` -r `)
```
cd /usr/local
mkdir mcpeserver
curl -sL https://get.pmmp.io | bash -s - -r
./start.sh
```
第一次使用会设定基本项，之后可以编辑server.properties修改设定。
由于服务器是阻塞进程，需要后台运行，方法是，mcpeserver.ini内容
```conf
[program:mcpeserver]
command=/usr/local/mcpeserver/start.sh
stdout_logfile=/var/log/mcpeserver/out.log
stderr_logfile=/var/log/mcpeserver/err.log
```
## pc服务器建服
如果出现
``` java
Exception in thread "main" java.lang.UnsupportedClassVersionError: net/minecraft/server/MinecraftServer : Unsupported major.minor version 52.0
```
的错误，可能是安装的java有问题(亲测，上述问题就是因为用的openjdk，换Oracle版的就好了)，请安装Oracle版的java(吐槽一句，阿里云下载java可能是做了本地缓存，1秒就下好了，但是文件又有问题，根本没办法安装)，Oracle java安装不上的请看[这篇文章](http://www.cnblogs.com/DebugLife/p/aliyun-ubuntu-java.html)
` Errors were encountered while processing:
 oracle-java7-installer
E: Sub-process /usr/bin/dpkg returned an error code (1)
 `
```bash
apt-get install python-software-properties
apt-get install software-properties-common
apt-add-repository ppa:webupd8team/java
apt-get update
sudo apt-get install oracle-java7-installer
cd /usr/local
mkdir minecraft_server
cd minecraft_server
wget https://s3.amazonaws.com/Minecraft.Download/versions/1.12/minecraft_server.1.12.jar
java -Xmx1024M -Xms1024M -jar minecraft_server.1.12.jar nogui
```
第一次运行会立即结束，修改eula.txt同意条约就行，第二次运行后修改server.properties，把online-mode从` true `改为` false `使盗版启动器也能登录。[官方wiki](http://minecraft-zh.gamepedia.com/index.php?title=Server.properties&variant=zh)写的超详细。
[使用supervisor]()管理进程，minecraft_server.ini配置如下
```conf
[program:minecraft_server]
command=/usr/java/java8/bin/java -Xmx1024M -Xms1024M -jar /usr/local/minecraft_server/minecraft_server.1.12.jar nogui
stdout_logfile=/var/log/minecraft_server/out.log
stderr_logfile=/var/log/minecraft_server/err.log
```
启动器都是使用的别人的，侵删，等我有时间了整一个。
