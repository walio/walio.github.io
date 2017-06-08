---
title: python每日进步
tags:
  - python
keywords:
  - python learning
  - urllib2和requests对比
categories:
---
urllib2.urlopen和requests.get对比：
1. 功能一样。urllib2.urlopen是异步调用，requests.get是同步调用，一旦有一个url阻塞住就无法继续往下了。grequests是异步的requests，但是无法取法取得响应信息。
2. 自定义404界面requests.get可以正常返回页面信息，status_code属性是404等，而urllib2抛出HTTPError异常
