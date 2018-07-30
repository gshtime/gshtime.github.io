---
title: git 使用指南
tags: git
date: 2018-05-18 00:11:03
---

# git 回滚

``` bash
# 回退命令：

$ git reset --hard HEAD^         回退到上个版本
$ git reset --hard HEAD~3        回退到前3次提交之前，以此类推，回退到n次提交之前
$ git reset --hard commit_id     退到/进到 指定commit的sha码

# 强推到远程：

$ git push origin HEAD --force
```

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
# git push -u <远程主机名> <本地分支名>  # 首次推送
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

# git diff 

用于比较两次修改的差异

``` bash
# 比较工作区与暂存区
git diff 不加参数即默认比较工作区与暂存区

# 比较暂存区与最新本地版本库（本地库中最近一次commit的内容）

git diff --cached  [<path>...] 

# 比较工作区与最新本地版本库

git diff HEAD [<path>...]  如果HEAD指向的是master分支，那么HEAD还可以换成master

# 比较工作区与指定commit-id的差异

git diff commit-id  [<path>...] 

# 比较暂存区与指定commit-id的差异

git diff --cached [<commit-id>] [<path>...] 

# 比较两个commit-id之间的差异

git diff [<commit-id>] [<commit-id>]

```

## 使用git diff打补丁

``` bash
git diff > patch  # patch的命名是随意的，不加其他参数时作用是当我们希望将我们本仓库工作区的修改拷贝一份到其他机器上使用，但是修改的文件比较多，拷贝量比较大，

# 此时我们可以将修改的代码做成补丁，之后在其他机器上对应目录下使用 git apply patch 将补丁打上即可

git diff --cached > patch  # 是将我们暂存区与版本库的差异做成补丁

git diff --HEAD > patch  # 是将工作区与版本库的差异做成补丁

git diff Testfile > patch  # 将单个文件做成一个单独的补丁

# 拓展：git apply patch 应用补丁，应用补丁之前我们可以先检验一下补丁能否应用，git apply --check patch 如果没有任何输出，那么表示可以顺利接受这个补丁
```
另外，可以使用git apply --reject patch将能打的补丁先打上，有冲突的会生成.rej文件，此时可以找到这些文件进行手动打补丁　