---
title: linux 命令行和 iterm2 操作指南
date: 2018-05-31 12:39:47
tags: linux command line, iterms, shortcut
---

# 标签

- 新建标签：`command + t`
- 关闭标签：`command + w`
- 切换标签：`command + 数字` 或 `command + 左右方向键`
- 切换全屏：`command + enter`

# 分屏

- 垂直分屏：command + d
- 水平分屏：command + shift + d
- 切换屏幕：command + option + 方向键 command + [ 或 command + ]

# 命令编辑

## 删除

- 删除单词：ctrl + w, 删除光标之前的单词
- 删除行：ctrl + u，删除到行首
- 删除行：ctrl + k，删除到行尾
- 删除字符：删除当前光标`ctrl + d`，删除光标之前：`ctrl + h`
- 清屏1：command + r
- 清屏2：ctrl + l

## 光标移动

- 光标到行首：ctrl + a (同 ⌘ + ←)
- 光标到行尾：ctrl + e (同 ⌘ + →)
- 单词移动：alt + b / alt + b 按单词前移/后移
- 字符移动：ctrl + f / ctrl +b 按字符前移/后移，相当于左右方向
- 交换光标处文本：ctrl + t

## 历史命令

- 查看历史命令：`command + ;` （输入打头几个字母，然后输入 `command + ;` iterm2将自动列出之前输入过的类似命令。）
- 查看剪贴板历史：command + shift + h
- 上一条命令：ctrl + p
- 搜索命令历史：ctrl + r

# 选中即复制

iterm2 有 2 种好用的选中即复制模式。

- 一种是用鼠标，在 iterm2 中，选中某个路径或者某个词汇，那么，iterm2 就自动复制了。 　　
- 另一种是无鼠标模式，`command + f`,弹出 iterm2 的查找模式，输入要查找并复制的内容的前几个字母，确认找到的是自己的内容之- 后，输入 tab，查找窗口将自动变化内容，并将其复制。如果输入的是 s- hift+tab，则自动将查找内容的左边选中并复制。
