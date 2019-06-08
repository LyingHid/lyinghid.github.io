---
layout: post
title:  "Hello Jekyll"
date:   2019-06-08 15:07:22 +0800
categories: jekyll hello-world
---

一直想找一个地方写博客，把自己的随想记下来。找来找去，发现GitHub和Jekyll的组合最适合我。我只需要使用markdown写博文，然后让Jekyll将markdown转换成网页，托管到GitHub上。

# 安装Jekyll
决定之后，第一步是将Jekyll安装到系统中。我用的系统是Arch Linux，所以参照Arch Wiki安装了相关的依赖包。

```shell
sudo pacman -S ruby rubygems
gem install jekyll
```

从上面的Shell命令能够看出Jekyll是一个ruby应用。然后ruby有自己的包管理器gem，gem对于ruby就好象pacman对于Arch linux一样。因为装这个东西就是为了写博客，所以具体gem和ruby是啥就没深究下去了。

安装完成后，Jekyll并不能直接被调用，它还不在环境变量中，添加一下就好了。

```shell
PATH="$PATH:$(ruby -e 'puts Gem.user_dir')/bin"
```

# 初始化博客目录
网上找了很多beginners' guide，照着[这个](https://3kni.com/2016/make-own-blog-with-jekyll-and-github-page/)我成功搭建好了。

首先要执行下面的命令，在Blog目下初始化一个博客。

```shell
jekyll new Blog
```

然后进到Blog目录下面，执行

```shell
cd Blog
jekyll serve
```

刚才创建的博客就可以在本地被浏览器访问了，terminal的输出告诉我们网址在[http://127.0.0.1:4000/](http://127.0.0.1:4000/)这个地方。打开网址后，可以发现Jekyll在创建博客的时候，默认为我们创建了一篇博文，

>![first look]({{site.baseurl}}/assets/imgs/first_look.png)

这篇初始博文告诉我们它被放在Blog/\_posts中，可以仿造它写新的博文。这篇文章就是仿造它写的，在写的时候，博客的本地服务器也在运行中，Jekyll会自动在我们对文章作出改动后，更新网站的内容，刷新一下浏览器就能看到更新后的样子了。

# GitHub托管博客
在本地访问博客，确认博客初始化成功后，就可以通过git上传到GitHub上去了。

我们需要先在GitHub上创建一个仓库，这个仓库名是有要求的，需要被设置为ACCOUNT_NAME.github.io。仓库名中的ACCOUNT_NAME就是我们自己的GitHub账户名。仓库名也是我们博客的网址，通过访问这个网址，就能看到我们写的博客啦。

# 个性化
现在看到的博客，博客中的名称，联系方式等元数据都是Jekyll初始化时帮我们填的，这些元数据在Blog/\_config.yml中，按照文件的引导修改一下就是我们自己的了。

再就是博客皮肤的事情了。嗯，默认皮肤现在看起来OK，就不折腾啦。
