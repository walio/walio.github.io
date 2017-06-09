---
title: 搭建一个没什么卵用的博客（三）瞎整编辑器
tags:
keywords:
  - atom
  - snippet
  - 代码片段
  - scope
  - timestamp
  - atom使用代码片添加时间
categories: 如何搭建一个舒适的博客创作环境
---
${/fillDataText}
**施工中**
<!-- more -->
atom介绍
首先例行公事吹一波atom。
>这个世界上有那么多种编辑器，为什么你要花时间学习和使用 Atom 呢？
虽然 Sublime 和 TextMate 之类的编辑器已经非常好用了，但它们仅提供了很有限的拓展性。而在另一个极端，Emacs 和 Vim 提供了灵活的拓展性，但它们并不是很友好，需要使用专用的编程语言来配置和拓展。
我们觉得我们可以做得更好。我们的目标是在保证易用性的同时提供充分的可拓展性（hackability）：这个编辑器会受到第一天学习编程的新生欢迎，而且当他们成长为编程专家时也难以割舍。
当我们使用 Atom 来开发 Atom 的时候，随着它的逐渐完善，我们愈发觉得已经离不开它了。从表面上来看，Atom 是一个能满足你的期待的，现代化的桌面文本编辑器，而在表面之下，这是一个值得你去一同完善的系统。

当然我们可以把它变得好用。
每次新建md文件都写很长的文件头title、tags内容是不是很烦？
可以通过创建代码片可以省略重复工作，只输入一小段单词即可自动补全。当然了你也可以切出去换命令行 ` hexo new <blog_name>`。

网上用Alt+Shift+S打开所有可用snippets都是骗人的，要配置快捷键才行。关于创建代码段网上很多讲的不是很详细
按下Ctrl+Shift+P打开控制面板，输入snip在下拉菜单中可以找到`Application: Open Your Snippets`项，打开snippets.cson文件，文件用CoffeeScript写就，有一段被注释了的实例代码：
```CSON
'.source.js':
    'console.log':
        'prefix': 'log'
        'body': 'console.log(${1:"crash"});$2'
```
网上很多教程大多是照这个改，很容易混淆的一点是，`.source.js`中js并不是指后缀名而是作用域，所以改为如下内容是没有用的：
```CSON
'.source.md':
    'blog head':
        'prefix': 'head'
        'body': """
          ---
          title: $1
          tags: $2
          keywords: $3
          categories: $4
          ---
        """
```
而应该是
```CSON
'.source.gfm':
    'blog head':
        'prefix': 'head'
        'body': """
          ---
          title: $1
          tags: $2
          keywords: $3
          categories: $4
          ---
        """
```
顺便提一句关于atom的代码作用域(scope)，关于这个我也没找到什么说的特别清楚的文章，去看已安装插件中的language-xxx插件，里面会写这个这个作用域的有关信息，比如我们所用到的language-gfm包，如snippet_scope.png中所示，
```
Scope: source.gfm
File Types:markdown, md, mdown, mkd, mkdown, rmd, ron
```
表示的是以markdown, md, mdown, mkd, mkdown, rmd, ron为后缀名的文件均属于`source.gfm`作用域，在`source.gfm`作用域上定义的代码片作用于以上所有后缀名。
顺带一提多个tags和keywords应该如下写
```markdown
keywords:
  - sqlmap
  - sqlmapapi
  - 匿名上网
```
而不是
```markdown
keywords: sqlmapapi sqlmap 匿名上网
```
需要插入date，hexo new出来的是带date的，不带date hexo会按修改时间排序？
关于snippets中时间变量和文件路径变量的问题关注[这个issue](https://github.com/atom/snippets/pull/173)，目前为止最后一次提交是2015年9月21日，尚未与master分支合并，强行安装需要atom的开发者模式。
sync同步插件
给出我的gist
