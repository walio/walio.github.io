---
title: 搜索引擎优化
tags: seo
keywords:
  - seo
  - 搜索引擎优化
  - sitemap
  - google search console
  - 百度站长工具
  - 360站长工具
  - next主题优化
categories: 如何整一个没什么卵用的博客
---
写了博客虽然没什么用，但还是希望能被人搜索到，被更多人看到的。虽然不是顶级域名，但还是可以稍微整一整SEO的。
<!-- more -->
1. SEO优化。首先用谷歌搜索site:walio.github.io，没有结果说明没被收录。[官方文档](http://theme-next.iissnan.com/third-party-services.html#google-webmaster-tools)写的很详细。[google search console](http://zhanzhang.baidu.com/dashboard/index)和[百度站长工具](http://zhanzhang.baidu.com/dashboard/index)及[360站长平台](http://zhanzhang.so.com/index.php?s=/Site/index.html)类似，添加方式相同。
2. 生成站点地图。安装hexo-generator-baidu-sitemap和hexo-generator-sitemap，生成了两个站点地图，记得要把站点配置文件的_config.yml中的url属性改了，否则默认生成的baidusitemap.xml和sitemap.xml中网站的地址都是http://yoursite.com/，谷歌提交站点地图会出现“
URL not allowed
This url is not allowed for a Sitemap at this location.”的错误。至于添加百度seo的时候一定要选对是https协议，否则也是一直无法验证，报301的错误。但是百度一直没能通过https验证，很难受。
3. 为hexo每篇文章添加关键字。关键字就是网页head中的meta标签，设定准确地关键字有利于搜索引擎搜索相关关键词时排名更靠前。默认next主题的文章关键字取文章的标签，所以如果想要设置很全的关键字，就需要设置很多tags，看着过于杂乱，通过在front-matter中加入keywords可解决。
4. 使用[谷歌数据分析](http://theme-next.iissnan.com/third-party-services.html#analytics-google)。详见[本文](google_ayalytics.md)。
