---
title: hexo 使用记录
date: 2018-05-18 11:03:14
tags:
---

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

