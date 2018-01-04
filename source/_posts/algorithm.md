---
title: 关于算法
date:
tags:
  - 算法
keywords:
  - algorithm
  - 算法
categories:
---
刷leetcode中的一些感想。虽然大部分都直接写在leetcode的Notes里了，但有些系统的成体系的算法还有值得单独写一写。
<!-- more -->
## Majority Voting Algorithm
[referer](https://gregable.com/2013/10/majority-vote-algorithm-find-majority.html)
[problem](https://leetcode.com/problems/majority-element-ii/description/)
second pass的作用是，有些数组根本不存在主元素，去除这些Corner case
其中candidate的意义其实应该是，candidate再最多“覆盖”count个元素，即，最多count个元素后，当前的candidate还不变。
