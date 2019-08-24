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

# 数学公式

写博客需要用数学公式，搜索了一下实现方案，基本都是在博文中加入LaTex格式的公式，然后通过JavaScript渲染。[MathJax](https://www.mathjax.org)是推荐的渲染脚本。


嵌入这个脚本需要更改生成网页的模板，模板位于网站根目录的_layouts文件夹下。但是，我的根目录没有这个文件夹，所以推测有一个系统全局的_layouts，这个全局模板的位置根据jekyll serve打印的log找到了。在这个模板的基础上我插入了MathJax，并保存在了博客根目录下新创建的_layouts中。但是，通过这个模板生成的博客缺失了标题，说明本地找到的模板不正确。那么上官网再试试，使用了官网的[minima模板](https://github.com/jekyll/minima/blob/master/_layouts/post.html)后，生成的网页和原来是一致的。嗯，成功找到默认模板了。


之后，按照[MathJax Doc](https://docs.mathjax.org/en/latest/configuration.html)的说明，在模板的article标签上方插入了如下脚本和配置。


```html
<script type="text/x-mathjax-config">
    MathJax.Hub.Config({
        extensions: ["tex2jax.js"],
        jax: ["input/TeX", "output/HTML-CSS"],
        tex2jax: {
            inlineMath: [ ['$','$'] ],
            displayMath: [ ['$$','$$'] ],
            processEscapes: true
        },
        "HTML-CSS": { fonts: ["TeX"] }
    });
</script>
<script type="text/javascript" async
        src="https://cdn.mathjax.org/mathjax/latest/MathJax.js?config=TeX-AMS-MML_HTMLorMML">
</script>
```


再写了个复杂的数学公式测试了一下，渲染成功。


```TeX
L=\sum{a_{i}}-\frac{1}{2}\left(\sum_i{\sum_j{\alpha_{i}\alpha_{j}y_{i}y_{j}x_{i}x_{j}}}\right)
```


$$ L=\sum{a_{i}}-\frac{1}{2}\left(\sum_i{\sum_j{\alpha_{i}\alpha_{j}y_{i}y_{j}x_{i}x_{j}}}\right) $$
