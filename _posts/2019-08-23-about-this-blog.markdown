---
layout: post
title: 关于我的博客网站
date: 2019-08-23 15:28:34.000000000 +08:00
tags: 不知所云
---

>浏览网页时偶然发现一个自己感觉比较简洁、漂亮的技术博客网站：[OneV's Den](https://onevcat.com/)，想着也按照博主的样式也自己搭建一个自己的博客网站，用于记录一些自己想记录的东西。

首先，博客基于[Jekyll](https://jekyllrb.com/)博客框架来搭建，并采用[Vno - Jekyll](https://github.com/onevcat/vno-jekyll)作为博客主题。所以需要以下两步：

- 安装Jekyll博客的开发环境
- 配置Vno - Jekyll作为博客主题

### 安装Jekyll博客开发环境
Jekyll依赖Ruby语言环境，所以要先安装Ruby。我自己电脑（Mac OS）本身自带了Ruby，版本为：

```bash
$ ruby -v
ruby 2.3.7p456 (2018-03-28 revision 63024) [universal.x86_64-darwin18]
```

>根据[Jekyll官网](https://jekyllrb.com/)中介绍的“三大步”安装教程，可以很快搭建一个静态博客网站。

首先，我们执行下面的指令来安装一下Jekyll：

```bash
$ gem install bundler jekyll
```

很遗憾，我运行该指令到最后，它报了一个错误给我：

```bash
··· ···

Fetching: jekyll-sass-converter-2.0.0.gem (100%)
ERROR:  Error installing jekyll:
	jekyll-sass-converter requires Ruby version >= 2.4.0.
```

嗯... 很明显，我电脑自带的Ruby版本没满足Jekyll要求的版本（`Ruby version >= 2.4.0`）。还能怎么办？那就来升级一下Ruby的版本呗：

>咱对Ruby也不在行，所以就搜索一下“如何升级Mac OS中的Ruby版本”。参考了这篇文章：[Mac OS升级Ruby](https://www.jianshu.com/p/a575aff064e3) 把Ruby升级到了2.6.3版本。这篇文章中介绍的升级方法，核心就是使用Ruby的第三方版本管理工具[RVM](http://rvm.io/)来完成。

那么就先来安装RVM（官网也有介绍[如何安装RVM](http://rvm.io/)）：

```bash
$ curl -L get.rvm.io | bash -s stable
```

安装完成之后，执行下面的指令来使RVM立即生效：

```bash
source ~/.rvm/scripts/rvm
```

来检查一下RVM是否已经成功安装以及生效：

```bash
$ rvm -v
rvm 1.29.9 (latest) by Michal Papis, Piotr Kuczynski, Wayne E. Seguin [https://rvm.io]
```

好了，RVM安装完毕。我们来升级Ruby的版本，从Ruby官网了解到最新的稳定版本是`2.6.3`：
```bash
$ rvm install 2.6.3
... ...
ruby-2.6.3 - #generating default wrappers.......
ruby-2.6.3 - #adjusting #shebangs for (gem irb erb ri rdoc testrb rake).
Install of ruby-2.6.3 - #complete
```

看上面的提示，应该是成功安装了`2.6.3`版本的Ruby，我们来检验一下：

```bash
$ ruby -v
ruby 2.6.3p62 (2019-04-16 revision 67580) [x86_64-darwin18]
```

没错了，接下来可以继续安装我们的Jekyll：
                           
```bash
$ gem install bundler jekyll
```

这个安装可能会有点耗时，需要耐心等待。。。

### 使用`Vno - Jekyll`作为博客主题

上一章将Jekyll开发环境搭建好了，接下来可以结合[Vno - Jekyll](https://github.com/onevcat/vno-jekyll)主题来搭建自己的博客。很简单，根据[Vno - Jekyll主题使用介绍](https://vno.onevcat.com/2016/02/hello-world-vno/)中的“四部曲”走起来：

>**特别需要注意：**[Vno - Jekyll主题使用介绍](https://vno.onevcat.com/2016/02/hello-world-vno/)这里面Usage中的指令有错误的：<br />
>文中指令`bundler install`和`bundler exec jekyll serve`中**应该是`bundle`而不是`bundler`。** 因为我按照文中的介绍来执行指令，发现一直报错。我查了半天，才发现问题...

```bash
$ git clone https://github.com/onevcat/vno-jekyll.git your_site
$ cd your_site
$ bundle install
$ bundle exec jekyll serve
```

启动成功之后，访问 <a href="http://127.0.0.1:4000" target="_blank">http://127.0.0.1:4000</a> 就可以看到搭建好的博客页面。

>`bundle exec jekyll serve`指令之后，Jekyll默认监听的`127.0.0.0:4000`地址。所以只有 <a>http://127.0.0.1:4000</a> 才能访问到，即只能从本机电脑访问；如果需要从其他网络访问位置访问博客，需要加`--host`参数，让Jekyll监听的host为`0.0.0.0`。

```bash
$ bundle exec jekyll serve --host 0.0.0.0
Configuration file: /root/LUZHO211/_config.yml
            Source: /root/LUZHO211
       Destination: /root/LUZHO211/_site
 Incremental build: disabled. Enable with --incremental
      Generating...
                    done in 0.408 seconds.
 Auto-regeneration: enabled for '/root/LUZHO211'
    Server address: http://0.0.0.0:4000
  Server running... press ctrl-c to stop.
```

还可以可以使用`--port`参数拉来修改Jekyll监听的端口：

```bash
$ bundle exec jekyll serve --host 0.0.0.0 --port 8080
Configuration file: /root/LUZHO211/_config.yml
            Source: /root/LUZHO211
       Destination: /root/LUZHO211/_site
 Incremental build: disabled. Enable with --incremental
      Generating...
                    done in 0.419 seconds.
 Auto-regeneration: enabled for '/root/LUZHO211'
    Server address: http://0.0.0.0:8080
  Server running... press ctrl-c to stop.
```

使用`jekyll serve -h`来查看Jekyll启动服务时支持的其他启动参数。






