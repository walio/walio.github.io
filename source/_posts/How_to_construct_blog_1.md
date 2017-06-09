---
title: 部署博客
tags:
  - hexo
  - 瞎折腾系列
keywords:
  - hexo
  - github pages
  - seo
categories: 如何整一个没什么卵用的博客
---
闲来无事搭建了博客，只为有趣，结果费了很多事。一开始用了最流行的jekyll，但发现这个东西还是挺复杂，如果ruby不在你的技术栈里还是不要轻易尝试了。这篇博客随着我部署的过程逐渐变长，想着只是写给自己看的，写着写着变成了大而全的东西。于是索性编成一个教程，怕链接失效把原有的许多链接内容也抄了过来，如果有侵犯版权，请告知，必定删除。其实[hexo官方文档](https://hexo.io/zh-cn/docs/)写的已经很详细了。
<!-- more -->
其实大部分步骤都能百度到，文章目的是整理下搭建博客的流程。没有用域名还是因为没找到好的域名。。没有增加文章置顶还是因为需要修改hexo-generator-index的源码，但是travis构建的时候却没有效果，还是等待官方给出具体解决方案吧。
1. 首先你得有一个github账号，不用多说。新建一个工程名为your_github_name.github.io，把静态页面提交至master分支，github会自动在https://your_github_name.github.io/ 生成页面。
2. 安装git。[下载地址](https://git-scm.com/downloads)。
3. 安装nodejs。[下载地址](https://nodejs.org/)。
4. 安装hexo。` npm install -g hexo-cl `，这句话相当于在全局安装hexo-cl，-g是global的简写，默认npm是会将库文件安装至当前文件夹下的node-modules，需要用-g参数指定全局安装。
5. 初始化hexo。切换到workspace文件夹，执行以下命令初始化hexo并安装hexo所需要的库。
   ```cmd
   hexo init <folder>
   cd <folder>
   npm install
   ```
6. 本地调试。在`<folder>`文件夹下运行` hexo server `命令可以启动一个本地的服务器，打开[页面](http://localhost:4000/)可以本地调试，修改源文件刷新即可看见效果。命令` hexo generate `会在`<folder>`下生成一个public文件夹，存储了静态页面。
7. 同步至github。有两种手段。
   （1）使用命令` npm install hexo-deployer-git --save `安装hexo-deployer-git扩展，然后在<folder>文件夹下生成的__config.yml中修改。
   ```yaml
   deploy:
     type: git
     repo: https://github.com/walio/walio.github.io.git
     branch: master
  ```
   在<folder>文件夹下执行` hexo deploy `命令，应当会弹出git的图形界面要求输入用户名和密码。完成以后就会把生成的静态页面自动部署至githubpages了。
   （2）持续部署。毕竟每一次这样修改了之后都要同步两个分支太麻烦，用travis CI可以帮助我们同时同步。参考[这篇文章](http://www.jianshu.com/p/e22c13d85659)。GH_TOKEN写成GH-TOKEN结果查了半天都没搞明白，还是要注意拼写。
8. 修改` <folder> `文件夹下的__config.yml文件，看着改就行，主要是title、author、description等。
9. 添加作者邮箱。因为没有评论，总要有一个别人联系我的方式(~~并不会有人联系~~)。
10. 添加图标。[图片制作为ico图标](http://tool.lu/favicon/)
11. *optional* 更改主题。个人感觉hexo默认的主题landscape不是很好看。换了next主题，这是[官方示例](http://notes.iissnan.com/)。安利一波next主题，灵活性好，开放性强，贡献者多，对于我辈不会自己造轮子的前端小白来说是福音。同样[官方文档](http://theme-next.iissnan.com/getting-started.html)也很详细。[官方wiki](https://github.com/iissnan/hexo-theme-next/wiki/%E5%88%9B%E5%BB%BA%E5%88%86%E7%B1%BB%E9%A1%B5%E9%9D%A2)。可以从头至尾看一遍` theme\next\__config.yml `按需要修改。
12. *optional* 添加七牛云。如果不需要图片其实可以不用，个人感觉用大部分图用echarts画足够甚至更好
往往画图的时候心有余而力不足，就需要用到echats：1.插件(方便)2.手动(可能不优雅？但可以保持最新的echats)3.手动引入需要的模块
13. *optional* 安装echarts。(1)插件（方便）(2)手动（不优雅，但通过cdn能保持最新版echarts）。其余一些有趣的插件
[hexo插件官方列表](https://hexo.io/plugins/)
14. 添加谷歌日历。


如果你的travis-CI构建运行npm install的时候遇到了Invalid version: "<https://registry.npmjs.org/hexo-util/-/hexo-util-0.6.0.tgz>"的问题，有可能是你的package-lock.json出了问题。package-lock.json是根据package.json自动创建的，删掉会重新创建，不用担心会出错。
