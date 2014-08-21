---
layout: post
title: "Python Buildout"
date: 2014-08-19 22:05
comments: true
categories: python
---
虽然 Python 使用 setuptools 可以非常方便的安装 egg package，但是如果遇到在服务器上面部署 Python 项目，但是机器没有外网怎么办，有没有类似 Maven 一样的工具，可以把 Python 项目打包成一个带有依赖的项目呢？

当然，今天就说说 Buildout，这个就是你需要的。

###Installing Buildout

我们使用 Glances 做例子，功能类似 top。

[Glances](http://example.com/ "Title") is a cross-platform curses-based system monitoring tool written in Python.

{% highlight bash %}
mkdir demo
cd demo
wget http://downloads.buildout.org/2/bootstrap.py
touch buildout.cfg
python bootstrap.py
{% endhighlight %}

这样就生成了一个 virtualenv 里面有 bin，parts，eggs，develop-eggs 目录。在 bin 目录下生成了 buildout 脚本。没错，buildout 是基于 setuptools 开发的。

{% highlight bash %}
git clone https://github.com/nicolargo/glances
{% endhighlight %}

然后我们编辑 `bootstrap.py` 文件。

{% highlight bash %}
cat buildout.cfg
[buildout]
develop = glances
parts = test

[test]
recipe = zc.recipe.egg
eggs = glances
{% endhighlight %}

每个 buildout 都要有一个 parts 列表，也可以为空，但是 buildout 必须是存在的。develop 就指定了我们开发的项目代码是一个相对路径，当然 glances 是 clone [nicolargo](https://github.com/nicolargo "Title") 的。

recipe = zc.recipe.egg 就是把一些把下面的 egg 安装到 eggs 目录中。

然后运行 `./bin/buildout`。

最后在 `bin` 下面就能看见 `glances` 命令。

More Info About [Buildout](http://www.buildout.org/en/latest/ "Title")

Enjoy it!
