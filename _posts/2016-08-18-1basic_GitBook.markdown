---
layout: post
title: "GitBook 使用教程"
subtitle: "快速实现从安装到发布你的书籍"
date: 2016-08-18 09:31:00 +0800
author: "Roach"
catalog: true
tags:
    - GitBook
---

> GitBook 是一个基于 Node.js 的命令行工具，可使用 Github/Git 和 Markdown 来制作精美的电子书。

GitBook 支持输出多种文档格式：pdf、eBook、静态网页（默认输出）。既然可以输出静态网页，那肯定可以通过 [GitHub Pages](https://pages.github.com/) 变成静态网站展示在互联网上。

在这里必须要赞美一下 Linus Torvalds，不仅编写的 `Linux`，而且为了开发 `Linux` 内核时版本控制方便创造了 `Git`。而这两项工作每一项都属于造福人类的范畴。

## 你个人书籍的展示方式

最终，你自己编写的内容可以有两种方式发布到互联网上：通过`GitHub`服务 或 通过 `GitBook` 进行发布。

第一种，需要你有一个`GitHub`帐号，然后在`GitHub`上添加一个存在你的书籍的仓库，这里假设仓库名字为 **book**。

第二种，则是在 [GitBook](https://www.gitbook.com/) 注册一个帐号，按后 `add new book`.

这里，主要针对第一种方法进行介绍，这个方法的主要思想就是：在**book**仓库的非`gh-pages`分支编写你的书籍，然后生成静态网页；之后推送到本地的`gh-pages`分支，最后推送到远程的`gh-pages`分支有`GitHub`生成静态网站。**这里需要敲黑板**！

---

## Prerequisites

要继续看本 `post` 的内容，你要知道：

- `Git` 和其简单的命令
- `GitHub` 相关操作
- 安装了 `Linux` 发行版的电脑

虽然，Windows和Mac也可以但是我只在`Linux`上进行了发布。其实，只要安装对应软件就可以。

## 软件安装

#### 安装Node.js

因为 gitbook 基于 nodejs 嘛。

```$ sudo dnf install nodejs```

#### 安装gitbook

```
$ sudo npm install gitbook -g
$ sudo npm install gitbook-cli -g
```

`gitbook` 命令：

```
$ gitbook -h
Usage: gitbook [options] [command]
Commands:
build [options] [source_dir] 编译指定目录，输出Web格式(_book文件夹中)
serve [options] [source_dir] 监听文件变化并编译指定目录，同时会创建一个服务器用于预览Web
pdf [options] [source_dir] 编译指定目录，输出PDF
epub [options] [source_dir] 编译指定目录，输出epub
mobi [options] [source_dir] 编译指定目录，输出mobi
init [source_dir]   通过SUMMARY.md生成作品目录
Options:
-h, --help     output usage information
-V, --version  output the version number
```

---

## 初始化书籍目录

#### clone远程仓库

1. 登陆到Github，进入你创建的book仓库
2. 克隆仓库到本地： git clone git@github.com:/USER_NAME/book.git
3. 创建一个新分支： git checkout -b gh-pages，注意，分支名必须为gh-pages
4. 将分支push到仓库： git push -u origin gh-pages
5. 切换到主分支：git checkout master

#### 编写README 与 SUMMARY

编写完这两个文件，可以使用`gitbook`初始话你书籍的目录章节。

拿[《C++ 拾遗》](https://roachsinai.github.io/Cpp-learning-notes/)为例，

`README.md`中的内容是：

```
# Cpp-learning-notes

> C++拾遗

On the way to be a good cpp programmer.

## 项目地址

- 仓库：https://github.com/roachsinai/Cpp-learning-notes
- 在线阅读：http://roachsinai.github.io/gitbook[xq@roach Cpp-learning-notes]
```

`SUMMARY.md`中的内容是：

```
# Summary

* [Introduction](README.md)
* [数据类型](types/README.md)
    * [数组](types/array.md)
    * [数组为什么不能进行赋值](types/array_lvalue.md)
    * [const and static](types/const_static.md)
* [参数传递](arguments/README.md)
    * [const形参和实参](arguments/arguments_parameters.md)
    * [顶层const](arguments/toplevel_const.md)
* [类](class/README.md)
    * [成员函数](class/member_function.md)
    * [访问、继承说明符](class/access_control.md)
    * [虚函数表](class/vptr.md)
* [Programming tips](tips/README.md)
    * [传递字符串](tips/cstring_parameter.md)
    * [浮点数比较](tips/floats_compare.md)
* [结束](end/README.md)
```

#### 目录初始化

当`SUMMARY.md`创建完毕之后，我们可以使用`Gitbook`的命令行工具将这个目目录结构生成相应地目录及文件

```
$ gitbook init

$ ls
arguments  class  end  README.md  SUMMARY.md  tips  types

$ tree -L 2 -U
.
├── README.md
├── SUMMARY.md
├── tips
│   ├── README.md
│   ├── cstring_parameter.md
│   └── floats_compare.md
├── end
│   └── README.md
├── types
│   ├── README.md
│   ├── array.md
│   ├── array_lvalue.md
│   └── const_static.md
├── arguments
│   ├── README.md
│   ├── arguments_parameters.md
│   └── toplevel_const.md
└── class
    ├── README.md
    ├── vptr.md
    ├── access_control.md
    └── member_function.md

5 directories, 17 files
```

可见，初始化产生的目录或者文件和`SUMMARY.md`中是对应的，然后可以去每个文件中编写你的内容。

---

## 输出静态网页

当编辑好你的内容之后，就可以使用`gitbook`命令来生成静态网页或者进行静态网站的本地预览（这时就已经生成了静态网页）。那就直接进行预览好了：

```$ gitbook serve [book_dir]```

`book_dir`是的`gitbook`书籍路径，这会在`book_dir`下生成一个`_book`文件夹，里面就是静态网站。

并且，`gitbook`会启动一个服务器监听`4000`端口用于预览：http://localhost:4000

#### 个性化

在你的书籍目录（仓库目录下）可以添加`book.json`，在其中添加或者修改内容可以实现：添加评论插件、添加书籍的信息、添加你的博客信息等

本`post`下方有`book.json`栗子~

但是，这时需要注意的是如果你选择了添加插件，那么需要执行

```gitbook install```

如果在初始化目录前，已经添加`book.json`，那么也要在`gitbook init`之前安装插件。

---

## 发布到Github Pages

到此，就知道只要把生成的静态网页推送到你`GitHub`仓库的`gh-pages`分支上，然后就可以在`https://github.com/USER_NAME/book`访问你的书籍了！

当然，你可以通过`git`命令来实现这些操作，不过像这种重复性的操作肯定已经有简单的解决方法了。

#### 使用gulp-gh-pages工具

###### 安装gulp

你是否`sudo`最好和本`post`一致 :-)

```
sudo npm install --global gulp-cli
npm init
npm install --save-dev gulp
```
 
###### 安装gulp-gh-pages

```npm install --save-dev gulp-gh-pages```
 
然后在你的目录中添加文件`glupfiles.js`，内容如下：
 
```
var gulp = require('gulp');
var ghPages = require('gulp-gh-pages');
 
gulp.task('deploy', function() {
  return gulp.src('./dist/**/*')
    .pipe(ghPages());
});
```

然后执行`$ gulp deploy`，之后就一键提交到`GitHub`上了，可以在互联网上访问你的书籍了。

访问地址：`https://github.com/USER_NAME/book`

类似工具还有：`grunt-gh-pages`

注：`$ gulp deploy`命令只是将静态网页提交到你项目的`gh-pages`分支上，`master`分支还是需要自己提交。

---

## book.json

贴一下`book.json`内容

```
{
    "title": "C++拾遗",
    "description": "C++学习笔记",
    "keywords": "C++, 学习笔记, markdown, gitbook",
    "introduction": {
        "path": "README.md",
        "title": "学习就是踽踽前行"
    },
    "plugins": [
        "mathjax", "disqus"
    ],
    "pluginsConfig": {
        "fontSettings": {
        "theme": "white",
        "family": "serif",
        "size": 2
        },
        "mathjax":{
            "forceSVG": true
        },
        "disqus": {
            "shortName": "roachsinai"
        }
    },
    "links": {
        "home": false,
        "about": false,
        "issues": false,
        "contribute": false,
        "sidebar": {
            "Roach's Blog": "http://roachsinai.github.io/"
        },
        "tail": {
            "C++拾遗": "http://roachsinai.github.io/Cpp-learning-notes/"
        },
        "gitbook": false,
        "sharing": {
            "google" : true,
            "facebook" : true,
            "twitter" : true,
            "weibo" : true,
            "qq" : true,
            "qrcode" : true
        }
    },
    "pdf": {
        "toc": true,
        "pageNumbers": true,
        "fontSize": 12,
        "paperSize": "a4",
        "margin": {
            "right": 62,
            "left": 62,
            "top": 36,
            "bottom": 36
        }
    }
}
```

---

## References

1. [Gitbook 使用入门](https://tonydeng.github.io/gitbook-zh/gitbook-howtouse/index.html)
2. [GitBook 简明教程](http://www.chengweiyang.cn/gitbook/index.html)
3. [GitBook, Git + Markdown 快速发布你的书籍](http://leeluolee.github.io/2014/07/22/2014-07-22-gitbook-guide/)
4. [glup Getting Started](https://github.com/gulpjs/gulp/blob/master/docs/getting-started.md)
5. [gulp-gh-pages](https://www.npmjs.com/package/gulp-gh-pages)