---
title: 搭建mc服务器(pc版和pe版)
date:
tags:
  - MineCraft
keywords:
  - Minecraft Server
  - 搭建mc服务器
  - 搭建mcpe服务器
categories:
---

| 客户端 | 版本 | 版本特性 | 下载链接 |
| ------| ------ | ------ | ------ |
| mc版本 | v1.12 | [链你妹的接啊](http://minecraft-zh.gamepedia.com/1.12) | [下载]() |
| mcpe版本 | v | [再链一个试试]() | [下载]() |
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
` Exception in thread "main" java.lang.UnsupportedClassVersionError: net/minecraft/server/MinecraftServer : Unsupported major.minor version 52.0 `
的错误，可能是安装的java有问题，请安装Oracle版的java
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
第一次运行后需要编辑server.
