---
layout: post
title: "在github上部署Octopress"
date: 2013-07-25 18:19
comments: true
categories: 
---

记录一下自己部署Octopress的经历吧。

## 目录
- [创建github repository](#cj)
- [安装Octopress](#az)
- [部署到github](#bs)
- [发布Post和换电脑](#post)

<a name="cj" id="cj"></a>
## 创建github repository
github的pages服务包括2种：User/Orgonization和Project。我做的是前者。

要创建一个名为“user.github.io”的repo，这样能保证可以通过`http://user.github.io`来访问页面。

<a name="az" id="az"></a>
## 安装Octopress
首先要保证Git和ruby正确安装。我在尝试Windows上安装时，用了ruby2.0，最后不能用，也不知道是不是不支持ruby2.0。反正官方的文档用的是1.9.3。

现在引用官方的文档：

先是下载octopress：
```
git clone git://github.com/imathis/octopress.git octopress
cd octopress    # If you use RVM, You'll be asked if you trust the .rvmrc file (say yes).
ruby --version  # Should report Ruby 1.9.3
```
然后安装依赖：
```
gem install bundler
rbenv rehash    # If you use rbenv, rehash to be able to run the bundle command
bundle install
```
最后安装默认的主题：
```
rake install
```

到这里，Octopress就安装完成了。在安装过程中，`gem install bundler` 和 `bundle install` 会十分耗时。需要做一点点修改来解决这个问题。

### 修改gem源
用 `gem sources -l` 看当前的源，默认是 `https://rubygem.org/`。

可以用 `gem sources --remove https://rubygem.org/` 把它删掉。然后用 `gem sources -a http://ruby.taobao.org/` 添加新源。看域名就知道是淘宝的，看来淘宝对中国的开源事业还是做了一定贡献的。

### 修改Gemfile
打开`octopress/Gemfile`，把第一行改成 `source "http://ruby.taobao.org"`

速度问题就此解决。

<a name="bs" id="bs"></a>
## 部署到github
直接用下面命令setup
```
rake setup_github_pages
```
这个命令会让你设置repo的路径，也就是`https://github.io/user/user.github.io`。至于作用，我就不翻译了，反正什么都给配好了。原文在[这里](http://octopress.org/docs/deploying/github/)。

<a name="fb" id="fb"></a>然后用下面的命令直接就发布了
```
rake generate
rake deploy
```

`rake deploy` 会把编译好的最终结果提交到repo的master分支。

<a name="bc" id="bc"></a>为了保存源码，还需要做点事情：
```
git add .
git commit -m 'your message'
git push origin source
```
这样就把源码提交到了repo的source分支。

<a name="post" id="post"></a>
## 发布Post和换电脑
用下面命令创建一篇新的Post：
```
rake new_post["title"]
```
这里`title`随便写(如果`title`里有中文，它还会自动转成拼音做文件名，真强)。Octopress会自动创建一个文件，并给出路径。只要用任何文本编辑器打开这个文件，根据MD语法写Post就行了。

写完后，需要再走一遍[发布](#fb)流程。同样，需要再次[保存源码](#bc)。

换了电脑后，首先要clone`user.github.io`这个repo。但是这样是无法deploy的。也没在官方文档里找到什么好办法。幸好在网上搜到了一种比较暴力的方法：

首先当然是要把当前分支切换到`source`。
```
git checkout source
```
然后创建`_deploy`目录，如果已经有了，就先删除，然后再创建。

进入`_deploy`目录，把master分支pull下来
```
git init
git remote add origin git@github.com:user/user.github.io.git
git pull origin master
```
这样一来就可以写新Post并且发布了。
