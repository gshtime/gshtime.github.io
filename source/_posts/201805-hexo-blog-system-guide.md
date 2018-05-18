---
title: hexo 使用记录
date: 2018-05-18 11:03:14
tags:
---
# 可视化写博客

借助vsc、atom之类的编辑器，可以实现hexo博客的编辑和实时预览，还可以试试 `hexo-admin`这款插件。

`hexo-admin` 能够管理文章，添加分类和标签，也可以一键部署到pages,现在图片可以实现粘贴上传，原插件为保存到`source/images`目录下,部署博客时同时上传。

另外，还有一款`hexo-admin-qiniu` 插件，实现了自动上传文件到七牛云的配置，比较方便。（不需要先安装`hexo-admin`，直接装这个就行了）

网址： [hexo-admin-qiniu github](https://github.com/xbotao/hexo-admin-qiniu)

# hexo 主题

## Next

Next是Hexo一个精简的主题系统，包含多种外观（Schema）选择，“精于心，简于形”是Next的目标。

[Next主题主页](http://theme-next.iissnan.com/)

### 下载主题

``` bash
$ cd your-hexo-site
$ git clone https://github.com/iissnan/hexo-theme-next themes/next
```
###

设置 语言
编辑 站点配置文件， 将 language 设置成你所需要的语言。建议明确设置你所需要的语言，例如选用简体中文，配置如下：

`language: zh-Hans`


# hexo 使用过程中出现的错误

- hexo本地测试运行重启后页面空白,提示 : `WARN No layout: index.html`?

原因：从git hexo分支（存放hexo文件）把代码拉下来，之前的Next 主题被忽略了，就没拉下来，所以必须重新 git clone Next主题的仓库

``` bash
git clone https://github.com/iissnan/hexo-theme-next themes/next
```

