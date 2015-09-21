---
layout:     post
title:      "Hi, Roach's Blog"
subtitle:   "老师去参加HPLC后的一天"
date:       2015-09-22 01:05:00 +0800
author:     "Roach"
header-img: "img/post-bg-universe.jpg"
tags:
    - 生活
---

>  GithubPages  jekyll  fork

上面是这个博客基于什么，偶然邂逅 Roach's Blog 的肯定多少对它们有所了解，当然也少不了git。

21日（昨天），首先根据 阮一峰 的博文搭建了Demo：jekyll-demo. 试水之后，决定用GP实现自己的博客。这里CSDN的渣网速，对我作出这个决定有不小帮助。顺带学习一下**Markdown**。

GP就不作介绍了...
jekyll目的：可以在本地预览即将发布的博客，然后进行修改、debug。但是安装jekyll之前要先安装ruby，ruby-devel(fedora)，nodejs... 然后 gem install jekyll （更新：gem update [package name]）

然后可以使用$: jekyll serve实时查看本地博客的变化，浏览器输入http://localhost:4000。
fork就是把你喜欢的jekyll theme cut and copy。

对于博客的搭建需要感谢：[huxpro](https://github.com/huxpro)，李兴华，廖雪峰，阮一峰！

#### 更新

重装了fedora，安装jekyll真废劲；23相对与21(fedora)来说安装后少了一些开发包，所以$ sudo gem install jekyll 时添加了相关依赖；并且，paginate必须在_config.yml中显式声明：

```
_config.yml
gems: [jekyll-paginate]
paginate: 20
paginate_path: "page:num"
```

并且jekyll升级到2.0之后，每次我最新的post总是生成不了，后来发现新版本加入了 Time Zone（结果就是时间比美国的早，所以jekyll认为时间不对不给生成），所以如果 date 这个 variable 的格式为 YYYY-MM-DD HH:MM:SS 将会默认显示为 UTC 时间（而不是我所在的时区），这样生成网页时如果使用的是 Asia/Shanghai 这个时区就会导致文章的显示日期以及时间有所改变。

解决方法：
1. date 的格式需要包含时区信息，对于Asia/Shanghai 这个时区，则需要将 date 的格式改为 YYYY-MM-DD HH:MM:SS +0800；
2. _config.yml中添加 timezone: Asia/Shanghai +0800 没有成功，囧～～～

#### 2016-08-17更新

现在，使用的系统是`Fedora 24`，而`jekyll`期间也有更新。并且，因为`atom`编辑器可以预览`markdown`，之前将`jekyll`卸载了。

今天，想使用`Gitbook`所以重新安装了`jekyll`。谁曾想挺繁琐，记录下来。

安装步骤：

1. sudo dnf install ruby ruby-devel
2. sudo dnf install redhat-rpm-config
3. sudo gem install jekyll jekyll-paginate

卸载步骤：

1. sudo gem uninstall -aIx （卸载所有gems）
2. sudo dnf remove  ruby ruby-devel redhat-rpm-config

#### 解决安装时出现问题的网页

1. [jekyll Troubleshooting](https://jekyllrb.com/docs/troubleshooting/)
2. [jekyll installed but 'command not found'](http://stackoverflow.com/questions/19276621/jekyll-installed-but-command-not-found)
3. [g++ error:/usr/lib/rpm/redhat/redhat-hardened-cc1 No that file and directory](http://stackoverflow.com/questions/34624428/g-error-usr-lib-rpm-redhat-redhat-hardened-cc1-no-that-file-and-directory)

第`3`个网页是真真有帮助的，头两个并没有不看最好。按我的步骤来即可。jekyll 安装时必须 sudo.
