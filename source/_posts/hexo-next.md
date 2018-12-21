---
title: 博客部署过程
tags:
  - hexo
keywords:
  - hexo
  - github pages
  - seo
categories: 搭建博客
---
闲来无事搭建了博客，只为有趣，结果费了很多事。一开始用了最流行的jekyll，但发现这个东西还是挺复杂，如果ruby不在你的技术栈里还是不要轻易尝试了。这篇博客随着我部署的过程逐渐变长，想着只是写给自己看的，写着写着变成了大而全的东西。于是索性编成一个教程，怕链接失效把原有的许多链接内容也抄了过来，如果有侵犯版权，请告知，必定删除。其实[hexo官方文档](https://hexo.io/zh-cn/docs/)写的已经很详细了。
<!-- more -->
## 步骤
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
11. *optional* 更改主题。个人感觉hexo默认的主题landscape不是很好看。换了next主题，这是[官方示例](http://notes.iissnan.com/)。安利一波next主题，灵活性好，开放性强，贡献者多，对于我辈不会自己造轮子的前端小白来说是福音。同样[官方文档](http://theme-next.iissnan.com/getting-started.html)也很详细。[官方wiki](https://github.com/iissnan/hexo-theme-next/wiki/%E5%88%9B%E5%BB%BA%E5%88%86%E7%B1%BB%E9%A1%B5%E9%9D%A2)。这里下载也有学问。直接下载，每次更新覆盖无疑是最麻烦的，如果作为子模块引入，修改了config.yml之后父模块无法同步提交。[比较好的方法](https://github.com/iissnan/hexo-theme-next/issues/328)。可以从头至尾看一遍` theme\next\__config.yml `按需要修改。
13. *optional* 安装echarts。(1)插件（方便）(2)手动（不优雅，但通过cdn能保持最新版echarts）。
14. *optional* 取消sidebar自动弹出
15. [一些高级技巧](http://www.arao.me/)，[所有hexo官方插件](https://hexo.io/plugins/)，[图标参考](http://fontawesome.io/cheatsheet/)，[各种绚丽效果](http://shenzekun.cn/hexo%E7%9A%84next%E4%B8%BB%E9%A2%98%E4%B8%AA%E6%80%A7%E5%8C%96%E9%85%8D%E7%BD%AE%E6%95%99%E7%A8%8B.html)
[关于置顶文章的问题](https://github.com/iissnan/hexo-theme-next/issues/415)
## bugs
#### 文章排序杂乱
首先感谢[同学](https://github.com/ukari)发现并帮忙解决了这个问题。
hexo博客next主题是完全根据修改时间来给博文排序的，并没有提供类似于置顶等功能。详细讨论看[这里](https://github.com/iissnan/hexo-theme-next/issues/415)。可以通过[修改hexo-generator-index模块](http://www.netcan666.com/2015/11/22/%E8%A7%A3%E5%86%B3Hexo%E7%BD%AE%E9%A1%B6%E9%97%AE%E9%A2%98/)的方法魔改，但如果是通过travis-CI生成博客，node-modules一般都被gitignore，所以无效。个人还是等待next作者更新添加feature，我觉得希望还是比较大。
另外如果出现每次构建以后文章创建日期相同的情况，是因为网页文件都是重新构建的，可以修改下.travis.yml文件，先clone再push
#### 更新麻烦
next处于一直更新当中，如果每次都要重新下载覆盖实在麻烦。可以把`/theme/next/_config.yml`文件移动到`/source/_data/next.yml`，然后将next作为子模块引入就可以啦，每次更新的时候对子模块更新就行。否则：
1. 直接在/theme/next文件夹中git clone，然后用git pull更新：/theme/next下的.git文件夹无法上传，重新下载后还要在/theme/next中重新git init
2. 直接将next主题作为子模块引入：git push无法修改_config.yml文件，我的理解是，next是单独的模块，需要有单独的远程库，修改了_config.yml文件后，要不提交到next的库，要不fork一个库来提交，而无论哪种都是不符合本意的。git pull则没关系，因为这只是对本地文件做的修改。
如果你的travis-CI构建运行npm install的时候遇到了Invalid version: "<https://registry.npmjs.org/hexo-util/-/hexo-util-0.6.0.tgz>"的问题，有可能是你的package-lock.json出了问题。package-lock.json是根据package.json自动创建的，删掉会重新创建，不用担心会出错。
