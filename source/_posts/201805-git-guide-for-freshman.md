---
title: git 远程仓库使用指南
tags: git
date: 2018-05-18 00:11:03
---

# git branch 命令

``` bash
# 重命名分支
git branch -m [old name] [new name]
```

# 克隆 git 仓库

``` bash
git clone <remote_repo> -b <branch>
```

# 远程仓库

查看、添加远程分支

``` bash
# 查看远程分支
git branch -r 

# 添加远程仓库
git remote add origin http://192.168.36.10:10080/quantum_rng_testing/nist # 添加远程仓库 origin 为自设的名字, ”quantum_rng_testing/nist“ 为工程的目录
```

## 远程分支基本操作

拉取篇

``` bash
# 拉取远程分支并创建本地分支
# 方法一，使用该方式会在本地新建分支，并自动切换到该本地分支，并建立映射关系
git checkout -b 本地分支名 origin/远程分支名
# 方式二，在本地新建分支，不会自动切换到该本地分支x，需要手动checkout，不会建立映射关系
git fetch origin 远程分支名:本地分支名

# 放弃本地所有修改，强制拉取远程更新
# 对于本地的项目中修改不做保存操作（或代码改崩），可以用到Git pull的强制覆盖
git fetch --all
git reset --hard origin/master
git pull  # 可以省略
# git fetch 指令是下载远程仓库最新内容，不做合并 
# git reset 指令把HEAD指向master最新版本
```

``` bash 
# 从远程仓库抓取数据
git fetch [remote-name]
git fetch -p  # 在fetch之后删除掉没有与远程分支对应的本地分支
```

推送篇

``` bash
# 推送
git push [参数] <远程主机名> <本地分支名>:<远程分支名>
# [参数] -u 第一次推送的时候，可以将分支进行关联，以后只要 `git push` 就行了

# 推送本地所有分支
# 不管是否存在对应的远程分支，将本地的所有分支都推送到远程主机，这时需要使用–all选项。
git push --all origin

# 强制覆盖远程分支
# 方法一
git push origin develop:master -f # 就可以把本地的develop分支强制(-f)推送到远程master
# 方法二 
git checkout master 		# 切换到旧的分支 
git reset –hard develop 	# 将本地的旧分支 master 重置成 develop 
git push origin master –force 	# 再推送到远程仓库
```

删除篇

``` bash
# 删除远程分支
$ git push origin --delete master
# 等同于推送一个空的本地分支到远程分支
$ git push origin :master
# git remote show origin 状态为stale是远端已经删除的
git remote prune origin  # 可以将其从本地版本库中去除
```

## 远程分支高级知识

本地分支和远程分支建立映射关系（跟踪关系track）后，使用 `git pull` 或者 `git push` 时就不必每次都要指定从远程的哪个分支拉取合并和推送到远程的哪个分支。

``` bash
# 查看映射关系（注意是双v）
git branch -vv

# 建立映射关系
git branch -u origin/hexo
# 或者
git branch --set-upstream-to origin/hexo  # git branch --set-upstream-to=origin/<branch> hexo
# 或者
git push --set-upstream[-u] origin hexo

# 撤销映射关系
git branch --unset-upstream
```

注：
> 本地分支可以与远程不同名的分支建立映射关系

# Refs 

[Git branch upstream](https://blog.csdn.net/tterminator/article/details/78108550)