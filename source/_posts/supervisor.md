---
title: supervisor相关
date: 2017-06-14
updated: 2017-06-20
tags:
  - linux
keywords:
  - supervisor
categories:
  - Linux
---
当使用ssh连接到Linux服务器时，开启阻塞进程后，关闭ssh连接即会停止阻塞进程，这时需要将阻塞进程s作为后台程序管理，个人感觉管理进程最佳工具是supervisor。[官方文档](http://supervisord.org/index.html)
<!-- more -->
supervisor是python写成，且只支持python2。
## 安装
linux自带python2，安装请用` sudo pip install supervisor `
```
Supervisor requires Python 2.4 or later but does not work on any version of Python 3.  You are using version 3.5.2 (default, Nov 17 2016, 17:05:23)
    [GCC 5.4.0 20160609].  Please install using a supported version.
```
尝试使用apt-get install supervisor
## 配置
centOS安装后会生成/etc/supervisor/supervisord.conf并自动包含/etc/supervisor/conf.d文件夹下所有ini文件，ubuntu要手动生成：
```bash
mkdir /etc/supervisor
mkdir /etc/supervisor/conf.d
echo_supervisord_conf > /etc/supervisor/supervisord.conf
vim /etc/supervisor/supervisord.conf
```
将
```
;[include]
;files = relative/directory/*.ini  
```
改为
```
[include]
files = conf.d/*.ini
```
修改完成后启动supervisor时/etc/supervisor/supervisord.conf会自动加载，并且supervisord.conf包含的/etc/supervisor/conf.d下的任何ini文件都会自动加载。ini文件基本语法：
```conf
[program:usercenter]
directory = /home/leon/projects/usercenter ; 程序的启动目录
command = gunicorn -c gunicorn.py wsgi:app ; 启动命令，可以看出与手动在命令行启动的命令是一样的
autostart = true ; 在 supervisord 启动的时候也自动启动
startsecs = 5 ; 启动 5 秒后没有异常退出，就当作已经正常启动了
autorestart = true ; 程序异常退出后自动重启
startretries = 3 ; 启动失败自动重试次数，默认是 3
user = leon ; 用哪个用户启动
redirect_stderr = true ; 把 stderr 重定向到 stdout，默认 false
stdout_logfile_maxbytes = 20MB ; stdout 日志文件大小，默认 50MB
stdout_logfile_backups = 20 ; stdout 日志文件备份数
stdout_logfile = /data/logs/usercenter_stdout.log; stdout 日志文件，需要注意当指定目录不存在时无法正常启动，所以需要手动创建目录（supervisord 会自动创建日志文件）

; 可以通过 environment 来添加需要的环境变量，一种常见的用法是修改 PYTHONPATH
; environment=PYTHONPATH=$PYTHONPATH:/path/to/somewhere
```
## 启动
运行` supervisord `启动，运行supervisorctl进入supervisorctl的shell界面，可以执行以下命令
```bash
> status    # 查看程序状态
> stop usercenter   # 关闭 usercenter 程序
> start usercenter  # 启动 usercenter 程序
> restart usercenter    # 重启 usercenter 程序
> reread    # 读取有更新（增加）的配置文件，不会启动新添加的程序
> update    # 重启配置文件修改过的程序
```
