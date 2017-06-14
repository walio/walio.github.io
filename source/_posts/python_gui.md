---
title: Tkinter中踩过的坑
date:
tags:
  - python
keywords:
  - Tkinter
  - python
  - gui
categories:
---
Tkinter只适合做最简单的应用，但凡复杂一点还是用wxpython或者pyQt吧
<!-- more -->
任何耗时的操作(io等)均会造成GUI卡死，请把它们放在单独的进程中执行。
少用pack()，我根本不知道怎么用pack()对齐
