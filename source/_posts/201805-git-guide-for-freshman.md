title: git使用指南
tags: git
date: 2018-05-18 00:11:03
---

# 远程仓库

## git push 命令
git push命令用于将本地分支的更新，推送到远程主机。它的格式与git pull命令相似。
```
$ git push <远程主机名> <本地分支名>:<远程分支名>
```

## git 查看远程分支

查看远程分支
```
git branch -r 
```

## git 拉取远程分支并创建本地分支

### 方法一

使用如下命令：
```
git checkout -b 本地分支名 origin/远程分支名
```
使用该方式会在本地新建分支，并自动切换到该本地分支x

> 采用此种方法建立的本地分支会和远程分支建立映射关系。

### 方式二

使用如下命令：
```
git fetch origin 远程分支名x:本地分支名x
```

使用该方式会在本地新建分支x，但是不会自动切换到该本地分支x，需要手动checkout。

> 采用此种方法建立的本地分支不会和远程分支建立映射关系。

### 本地分支和远程分支建立映射关系的作用

ref: 博文[Git branch upstream](https://blog.csdn.net/tterminator/article/details/78108550)

#### 目标

本博文中git操作的目标为建立本地分支与远程分支的映射关系（或者为跟踪关系track）。这样使用 `git pull` 或者 `git push` 时就不必每次都要指定从远程的哪个分支拉取合并和推送到远程的哪个分支了。

#### 查看本地分支与远程分支的映射关系

使用以下命令（注意是双v）：

```
git branch -vv
```

可以获得如下信息： 
```
* hexo   d60a56a create the world
  master 649e511 [origin/master: ahead 6, behind 3] Site updated: 2018-04-22 18:23:40
```
可以看到分支hexo没有和远程分支建立任何映射，此时若执行如下拉取命令则不成功（因为git此时不知道拉取哪个远程分支和本地分支合并）： 
```
$ git pull
There is no tracking information for the current branch.
Please specify which branch you want to merge with.
See git-pull(1) for details.

    git pull <remote> <branch>

If you wish to set tracking information for this branch you can do so with:

    git branch --set-upstream-to=origin/<branch> hexo
```
同理，若此时执行如下推送命令同样不成功： 
```
$ git pull
There is no tracking information for the current branch.
Please specify which branch you want to merge with.
See git-pull(1) for details.

    git pull <remote> <branch>

If you wish to set tracking information for this branch you can do so with:

    git branch --set-upstream-to=origin/<branch> hexo

Kevin-MacBook-Air:blog arthur-mac$ git push
fatal: The current branch hexo has no upstream branch.
To push the current branch and set the remote as upstream, use

    git push --set-upstream origin hexo
```

#### 建立本地分支与远程分支的映射关系

建立当前分支与远程分支的映射关系:
```
git branch -u origin/hexo
```

或者使用命令：
```
git branch --set-upstream-to origin/hexo
```
得到结果如下： 
```
Branch 'hexo' set up to track remote branch 'hexo' from 'origin'.
```
查看当前本地分支与远程分支的映射关系结果如下： 
```
$ git branch -vv
* hexo   d60a56a [origin/hexo] create the world
  master 649e511 [origin/master: ahead 6, behind 3] Site updated: 2018-04-22 18:23:40
```
此时就能够正常的拉取和推送了。

#### 撤销本地分支与远程分支的映射关系
撤销本地分支与远程分支的映射关系

```
git branch --unset-upstream
```
使用git branch -vv得到结果如下： 
```
$ git branch -vv
* hexo   d60a56a create the world
  master 649e511 [origin/master: ahead 6, behind 3] Site updated: 2018-04-22 18:23:40
```
可以看到本地分支与远程分支的映射关系已经撤销。

#### 问题思考：本地分支只能跟踪远程的同名分支吗？

答案是否定的，本地分支可以与远程不同名的分支建立映射关系。