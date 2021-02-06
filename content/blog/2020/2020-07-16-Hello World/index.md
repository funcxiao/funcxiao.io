+++
title = "Hello World"
description = "First blog"
draft = false
[taxonomies]
tags = ["Others", "Zola"]
[extra]
feature_image = "overview.jpg"
feature = true
link = ""
+++


没啥好说的，重新弄个静态页面博客，静态页面生成器使用的是 [Zola](https://www.getzola.org/)， 页面主題使用的是 [Float](https://gitlab.com/float-theme/float)。
******

## 用到的工具

- zola,用来创建项目并且本地预览。
- vscode,用来编辑项目文件，顺便现在拿来写markdown也很好，也就不需要用typora了。
- github,用来存放项目模板和md文件，是整个工作的枢纽。
- [netlify](https://www.netlify.com),可以用来做持续集成工具，更进一步的就是把页面都给部署到netlify上去。
- [utterances](https://utteranc.es/)，利用 GitHub issue 作為留言系統，zola的主题`float`里面集成了。

## 简要流程

本地创建项目：

```Shell
# 创建文件夹myblog，Git初始化并新建zola项目
$ git init && zola init
# 添加Float主题，作为Git子模块加入
$ git submodule add https://gitlab.com/float-theme/float.git themes/float
```

編輯项目的 config.toml，给github仓库安装[utterances app](https://github.com/apps/utterances)用来留言 ：

```TOML
# 指定主题为float
theme = "float"

# 加入tag作为分类
taxonomies = [
    {name = "tags", paginate_by = 10},
]
# 添加留言配置
[extra]
utterances = true
utterances_repo = "xxxx/xxxx"
```

然后，使用最简单的办法就是编写一份[netlify.toml](https://www.getzola.org/documentation/deployment/netlify/),这样在netlify网站中添加github仓库后，每次本地写完了推送到github上，netlify就会拉过去使用配置文件中指定版本的的`zola`来生成静态网页并部署在netlify上，也可以部署到`github page`上(这样netlify就单纯作为一个CI使用了),当然作为懒人我选择都丢给netlify了。

完成本地文件编写之后，推送到github仓库就大功告成了，基本流程和使用`hugo`是差不多的，区别大概就是netlify预设好了对hugo的支持，使用zola则需要自己去设置，不过现在netlify比较强大，一份配置文件就搞定了。后续更新在项目的`content`目录下添加markdown文件并推送到ggithub就可以了。

## 拓展和问题

zola安装主题使用之后，还可以在项目的`templates`目录下编写额外的html文件，在外面`templates`目录下编写的模板文件可以替换掉`themes/xxxx`目录下主题相同路径名的模板。当然除了替换还可以选择继承主题下的模板，并在基础上进行修改：

```HTML
{% extends "theme_name/templates/page.html" %}
{% block title %}{{ page.title }}{% endblock %}
```

因为zola生成静态页面的时候，会对文件路径、文章标题等进行`slugify`操作（对中文而言，基本就是汉字转拼音，空格替换为'-'）然后生成id，所以在vscode中写markdown使用插件生成的`TOC`跳装对于中文或者一些特殊符号就失效了。不过可以使用zola内置的参数`page.toc`来实现。

## 别的

上一个用的生成静态页面的工具叫`hugo`，应该就是`雨果`，现在用的`zola`，应该就是`左拉`，都是文豪，以后可能还有起名叫`大仲马`的，可能已经有了，哈哈哈。
