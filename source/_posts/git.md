---
title: git
date:
tags:
keywords:
categories:
---
仅仅是给自己看的笔记
参考 [progit](https://git-scm.com/book/zh/v2)
<!-- more -->
git status查看本地和远程分支状况，提示很友好，告诉你达到怎样的效果需要什么命令。
## 首先不看远程分支。
首先git是不管文件具体变化的，只会记录哪些文件变化了，然后记录文件前后的内容。比较文件变化是别的工具做的，linux下可以指定vimdiff等，Windows的会自动显示diff应该是自带了？
本地分支有五个状态，untracked，unmodified，modified，staged，committed。commited状态其实就是unmodified状态。需要注意的是git add XXX并不是字面上看上去的把文件从untracked变为tracked，而是将目标文件（从tracked或者untracked状态）直接变为staged状态。简单来说，git add将文件变为staged，git commit将文件变为committed。但是git commit -a只会提交tracked的文件，而不会对untracked的文件做任何操作。想一下也能明白，git add的时候都需要添加参数，不指定的话自然untracked的文件对git来说是透明的。
![“git状态转移”](/img/git_status.jpg)
git stash：将当前工作区暂时存储起来，包括staged和unstaged的文件，untracked的文件会怎样？如何处理冲突？
5. git checkout branchname~2可能会使HEAD移动到特定提交但没有任何引用分支，此时所做的任何commit，在checkout到其它位置后都会丢失。
6. git revert会建立一个反向提交，用于“删除”指定提交到HEAD之间的修改，而不会改变提交历史。
7. checkout和reset的区别：git checkout -- filename使用staged区文件覆盖本地文件，而reset是使用committed区覆盖暂存区（如--hard则覆盖暂存区和工作区）
## 关于分支
HEAD指向现在正在工作的地方
`git branch XXX`新建分支但不会切换到新建分支上（即不会改变HEAD），`git checkout XXX`=`git branch XXX+git checkout XXX`，会切换到新建分支。
4. 分支合并：merge；rebase：一般来说和merge一样，只不过提交历史更清晰，不会有分支，也可以将其他分支中的矢量变化（部分改变）合并到本分支而不是所有改变。
## 引入远程仓库后
1. 本地和远程仓库同步时是同步所有信息，包括所有分支，提交历史，在任意一个本地分支上都能还原出整个项目的所有内容。对远程分支操作时不存在回退等操作，只需要在本地仓库回退然后push和远程仓库保持一致就可以了。
2. git clone实际操作时在本地创建master分支，然后将远程的master分支同步到本地master分支，所以直接git pull会看到远程有新的分支，但不会像clone那样自动创建分支并同步，本地没有分支与之同步，无法通过git branch branchname那样切换到新的分支，需要别的分支的话需要`git checkout -b branchname origin/branchname`，在本地新建分支并和远程同步（此处不需联网，实际所有分支和历史都下载到本地了，只是需要checkout出来），或者直接`git clone -b branchname url`克隆指定分支。
3. 删除远程分支的搞笑方法：git push origin :branchname。相当于将空分支推到远程分支。
3. git fetch取回远程文件，但是仅在.git中，不会修改本地文件，需要自行merge。git pull = git fetch+merge
