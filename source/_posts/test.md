---
title: 测试博客
tags: test
---

也算是闲来无事搭建了一个博客，费了很多事，一开始用了最流行的jekyll，发现这个ruby的东西很难搞定，如果不懂ruby的话还是不要轻易尝试好了。ruby也不在我准备学习的技术栈里，虽然整上但又放弃了。还是hexo看着更简洁，虽然部署的时候麻烦。总之记录下自己部署的过程吧。其实[hexo官方文档](https://hexo.io/zh-cn/docs/)写的已经很详细了。
<!-- more -->
1. 安装git。这是hexo官方文档给出的[链接](https://git-scm.com/downloads)。
2. 安装nodejs。[nodejs官网](https://nodejs.org/)去下对应的版本安装就好了。
3. 安装hexo。 ` npm install hexo-cl `
4. 初始化hexo。切换到你的workspace文件夹，执行以下命令安装nodejs所需要的库。
   ```cmd
   hexo init <folder>
   cd <folder>
   npm install
   ```
5. 更改主题。(optional)个人感觉hexo默认的主题[landscape](https://hexo.io/hexo-theme-landscape/)不是很好看，虽然还有[landscape-plus](http://xiangming.github.io/landscape-plus/)。换了[next](http://notes.iissnan.com/)的主题。同样[官方文档](http://theme-next.iissnan.com/getting-started.html)也很详细。很神奇的是默认语言是日语。需要根据[这里](http://theme-next.iissnan.com/getting-started.html#select-language)修改。
6. 本地调试。在`<folder>`文件夹下运行` hexo s `命令可以启动一个本地的服务器，打开[页面](http://localhost:4000/)就可以看到效果。hexo默认使用4000端口，如果无法启动可以试试查看端口是否被占用。修改了<folder>中的文件后刷新页面即可看到效果。命令` hexo generate `可以生成一个public文件夹，这个是静态网页，目标就是将这些静态网页上传至github，通过github pages访问。
7. 同步至github。使用命令` npm install hexo-deployer-git --save `安装hexo-deployer-git扩展，然后在<folder>文件加下生成的__config.yml中修改。
   ```yaml
   deploy:
     type: git
     repo: https://github.com/walio/walio.github.io.git
     branch: master
  ```
   在<folder>文件夹下执行` hexo deploy `命令，应当会弹出git的图形界面要求输入用户名和密码。完成以后就会生成githubpages了。
8. 持续部署。毕竟每一次这样修改了之后都要同步两个分支太麻烦，用travis CI可以帮助我们同时同步。[这篇文章](http://www.jianshu.com/p/e22c13d85659)已经写的很详细了，大家还是要注意拼写，我把下划线写成中划线结果查了半天都没搞明白。
9. 添加徽章。(optional)这个纯属自己消遣了应该。。。
10. SEO优化。写了博客虽然没什么用，但还是希望能被人搜索到，被更多人看到的。首先用百度搜索site:walio.github.io，没有搜到说明就没被收录了。[这篇文章](https://zonghongyan.github.io/2017/02/18/201702181911/)同样写的很详细。

还是很高兴地看到其中很多优秀的应用都是由国人所编写的。还有很多事情要做，比如把以前的笔记迁移过来，还要考虑怎么配置一个比较舒适的写笔记的环境，包括有时候可能要在平板或者手机上写。还在考虑要不要通过vps同步到github，毕竟这个环境搭起来也有点麻烦。但有可能过一阵我要换一个vps，现在这个没有办法通过vpn登陆，要做一些见不得人的事情还是不放心。有时间可能写一写自己玩vps的过程。
