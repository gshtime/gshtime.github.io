---
title: hexo 使用记录
date: 2018-05-18 11:03:14
tags:
---

# 安装

1. 安装Node.js

```
curl https://raw.github.com/creationix/nvm/master/install.sh | sh
```

2. 安装git

```
Windows：下载并安装 git.
Mac：使用 Homebrew, MacPorts ：brew install git;或下载 安装程序 安装。
Linux (Ubuntu, Debian)：sudo apt-get install git-core
Linux (Fedora, Red Hat, CentOS)：sudo yum install git-core
```

3. 安装Hexo

```
$ npm install -g hexo-cli
```
之后要在博客的文件夹下执行以下命令
```
npm install hexo
npm install
npm install hexo-deployer-git
```

# 可视化写博客

借助vsc、atom之类的编辑器，可以实现hexo博客的编辑和实时预览，还可以试试 `hexo-admin`这款插件。

`hexo-admin` 能够管理文章，添加分类和标签，也可以一键部署到pages,现在图片可以实现粘贴上传，原插件为保存到`source/images`目录下,部署博客时同时上传。

另外，还有一款`hexo-admin-qiniu` 插件，实现了自动上传文件到七牛云的配置，比较方便。（不需要先安装`hexo-admin`，直接装这个就行了）

网址： [hexo-admin-qiniu github](https://github.com/xbotao/hexo-admin-qiniu)

# hexo 主题

> 说明：在 Hexo 中有两份主要的配置文件，其名称都是 `_config.yml`。 其中，一份位于站点根目录下，主要包含 Hexo 本身的配置；另一份位于主题目录下，这份配置由主题作者提供，主要用于配置主题相关的选项。

为了描述方便，在以下说明中，将前者称为 站点配置文件， 后者称为 主题配置文件。
## Next

Next是Hexo一个精简的主题系统，包含多种外观（Schema）选择，“精于心，简于形”是Next的目标。

[Next主题主页](http://theme-next.iissnan.com/)

### 下载主题

``` bash
$ cd your-hexo-site
$ git clone https://github.com/iissnan/hexo-theme-next themes/next
```

### 启用主题

与所有 Hexo 主题启用的模式一样。 当 克隆/下载 完成后，打开站点配置文件 `_config.yml`， 找到 theme 字段，并将其值更改为 next。

```
# 启用 NexT 主题
theme: next
```
到此，NexT 主题安装完成。下一步我们将验证主题是否正确启用。在切换主题之后、验证之前， 我们最好使用 hexo clean 来清除 Hexo 的缓存。

主题设定包括：详细见官网介绍
- 选择「Scheme」
- 设置「界面语言」
- 设置「菜单」
- 设置「侧栏」
- 设置「头像」
- 设置「作者昵称」
- 设置「站点描述」

### 设置语言

编辑 站点配置文件， 将 language 设置成你所需要的语言。建议明确设置你所需要的语言，例如选用简体中文，配置如下：

`language: zh-Hans`


# hexo 使用过程中出现的错误

- hexo本地测试运行重启后页面空白,提示 : `WARN No layout: index.html`?

原因：从git hexo分支（存放hexo文件）把代码拉下来，之前的Next 主题被忽略了，就没拉下来，所以必须重新 git clone Next主题的仓库

``` bash
git clone https://github.com/iissnan/hexo-theme-next themes/next
```

