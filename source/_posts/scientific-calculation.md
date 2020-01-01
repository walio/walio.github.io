---
title: 科学计算库踩坑
date: 2020-01-01 22:05:59
updated: 2020-01-01 22:05:59
tags:
---
numpy、pandas、sklearn踩坑系列。
<!--more-->
1. numpy遇到0整除的情况。
numpy里面/0是不会报错只会报warning，而且0/0=0，0.0/0=nan，0.1/0=inf，容易在后续引起问题，可以设置np.seterr(all='raise')
2. np.exp(decimal)没问题，np.log(decimal)会报错。。。。。。。see https://github.com/numpy/numpy/issues/9955#issuecomment-451648429 (todo)
3. np中的数组应该是类似于矩阵，ndarray.sort(axis=1)只会改变第二列的顺序，第一列保持不变，这样是无法实现按照第一列内容对数组排序的
4. matplotlib中，画图的时候最终是依靠的axes而不是plt，即使是plt.show()也是调用了axes的方法，axes表示的是”轴域“的概念，plt.show()之类了的方法像是一种模糊不清的语法糖，当没有子图的时候，一般人都不会调用subplot方法得到axes再画图，而是直接plt.show()，但不得到axes无法进行较为精细的操作，例如set_major_formatter等（横轴为时间时，默认会）。如各个轴的样式、间隔等。通过plt.gca()可以得到当前的axes（get current axes）
5. 令人无语的numpy：整数的负整数次方无法计算，如np.array([1,2,3,4,5]) ** (-2)会报Integers to negative integer powers are not allowed的错误，因为numpy的输出值的dtype不是按照值来定的，而是按照输入来定的，这意味着np.array([1,2,3,4,5]) ** (-2)输出的dtype要不和np.array([1,2,3,4,5]) ** (2)一直，要不就不输出，而np.array([1,2,3,4,5]) ** (2) dtype是整数，而-2次方做不到，因此numpy选择不输出。这个逻辑太诡异了。。。。。refer：https://stackoverflow.com/questions/43287311/why-cant-i-raise-to-a-negative-power-in-numpy
