title: git 使用指南
tags: git
date: 2018-05-18 00:11:03
---

# 克隆 git 仓库

命令

``` bash
git clone <remote_repo> -b <branch>
```

# 远程仓库

## 添加远程仓库

``` bash
git remote add origin http://192.168.36.10:10080/quantum_rng_testing/nist # （”quantum_rng_testing/nist“ 为工程的目录）
```

## git push 命令

git push命令用于将本地分支的更新，推送到远程主机。它的格式与git pull命令相似。

```
$ git push [参数] <远程主机名> <本地分支名>:<远程分支名>
```
参数：
- -u 第一次推送的时候，可以将分支进行关联，以后只要 `git push` 就行了

### 删除远程分支
如果省略本地分支名，则表示删除指定的远程分支，因为这等同于推送一个空的本地分支到远程分支。
```
$ git push origin :master
# 等同于
$ git push origin --delete master
```

### 强制覆盖远程分支

``` bash
# 方法一
git push origin develop:master -f # 就可以把本地的develop分支强制(-f)推送到远程master

# 方法二 
git checkout master 		# 切换到旧的分支 
git reset –hard develop 	# 将本地的旧分支 master 重置成 develop 
git push origin master –force 	# 再推送到远程仓库
```

### 推送本地所有分支
不管是否存在对应的远程分支，将本地的所有分支都推送到远程主机，这时需要使用–all选项。
```
git push --all origin

```

## 放弃本地所有修改，强制拉取远程更新

开发时，对于本地的项目中修改不做保存操作（或代码改崩），可以用到Git pull的强制覆盖，具体代码如下：

``` bash
git fetch --all
git reset --hard origin/master
git pull //可以省略
```
git fetch 指令是下载远程仓库最新内容，不做合并 
git reset 指令把HEAD指向master最新版本


## 查看远程分支

查看远程分支
```
git branch -r 
```

## 拉取远程分支并创建本地分支

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
