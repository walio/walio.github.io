---
title: next主题一些技巧
date:
tags:
  - hexo
keywords:
  - hexo
  - next
  - hexo生成的博客创建时间都相同
categories: 如何整一个没什么卵用的博客
---
一些小技巧，同样是没什么卵用的
<!-- more -->
## 文章排序杂乱
首先感谢[同学](https://github.com/ukari)发现并帮忙解决了这个问题。
hexo博客next主题是完全根据修改时间来给博文排序的，并没有提供类似于置顶等功能。详细讨论看[这里](https://github.com/iissnan/hexo-theme-next/issues/415)。可以通过[修改hexo-generator-index模块](http://www.netcan666.com/2015/11/22/%E8%A7%A3%E5%86%B3Hexo%E7%BD%AE%E9%A1%B6%E9%97%AE%E9%A2%98/)的方法魔改，但如果是通过travis-CI生成博客，node-modules一般都被gitignore，所以无效。个人还是等待next作者更新添加feature，我觉得希望还是比较大。
另外如果出现每次构建以后文章创建日期相同的情况，是因为网页文件都是重新构建的，可以修改下.travis.yml文件，先clone再push
## 更新麻烦
next处于一直更新当中，如果每次都要重新下载覆盖实在麻烦。可以把`/theme/next/_config.yml`文件移动到`/source/_data/next.yml`，然后将next作为子模块引入就可以啦，每次更新的时候对子模块更新就行。否则：
1. 直接在/theme/next文件夹中git clone，然后用git pull更新：/theme/next下的.git文件夹无法上传，重新下载后还要在/theme/next中重新git init
2. 直接将next主题作为子模块引入：git push无法修改_config.yml文件，我的理解是，next是单独的模块，需要有单独的远程库，修改了_config.yml文件后，要不提交到next的库，要不fork一个库来提交，而无论哪种都是不符合本意的。git pull则没关系，因为这只是对本地文件做的修改。
