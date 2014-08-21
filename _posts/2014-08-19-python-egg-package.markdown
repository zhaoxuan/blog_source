---
layout: post
title: "Python egg"
date: 2014-08-19 22:05
comments: true
categories: python
---
对于用 Python 开发的同学，对 pip 和 easy_install 已经是非常熟悉了。easy_install 是由 PEAK(Python Enterprise Application Kit) 开发的 setuptools 包里带的一个命令，它用来安装egg包。

###Installing setuptools

    wget https://bootstrap.pypa.io/ez_setup.py -O - | python

如果使用了 pyenv 安装的 Python，setuptools 已经是安装好了。

###Making first egg package

一个 egg 主要包括一个 setup.py 文件。

{% highlight bash %}
mkdir egg_demo
cd egg_demo
touch setup.py
{% endhighlight %}

在 setup.py 文件里面，通过一个 setup() 函数来配置打包信息。

{% highlight python %}
#!/usr/bin/env python
#-*- coding:utf-8 -*-

from setuptools import setup

setup()
{% endhighlight %}

通过下面的 Python script 就能生成一个 egg 包。

    python setup.py bdist_egg

可以看到多了三个文件夹。而在 dist 文件夹下，有一个 egg 文件：UNKNOWN-0.0.0-py2.6.egg，这个 egg 就是我们最后的目标文件。

接下来我们修改 setup() 里面的参数。

{% highlight python %}
#!/usr/bin/env python
#-*- coding:utf-8 -*-

from setuptools import setup, find_packages

setup(
    name = "egg_demo",
    version="0.0.1",
    packages = find_packages('.'),
    zip_safe = False,

    description = "egg demo.",
    long_description = "egg demo, for testing.",
    author = "john",
    author_email = "xxx@qq.com",

    license = "GPL",
    keywords = ("test")
    )
{% endhighlight %}

我们再建立一个 egg_demo 文件夹，里面放我们需要的代码。

    mkdir egg_demo
    touch egg_demo/__init__.py

在 `__init__.py` 文件里面加入以下代码

{% highlight python %}
#!/usr/bin/env python
# -*- coding: utf-8 -*-
# @Author: john
# @Date:   2014-08-20 22:37:35
# @Last Modified by:   john
# @Last Modified time: 2014-08-20 22:38:43

def say_hello():
    print "hello work"
    pass
{% endhighlight %}

再次生成 egg 包。马上就能通过 setuptool 来安装我们自己写的包了。

{% highlight bash %}
python setup.py install

running install
running bdist_egg
running egg_info
...
Installed /Users/john/.pyenv/versions/py276/lib/python2.7/site-packages/egg_demo-0.0.1-py2.7.egg
Processing dependencies for egg-demo==0.0.1
Finished processing dependencies for egg-demo==0.0.1
{% endhighlight %}

通过 python 来试试我们的 first egg。

{% highlight python %}
import egg-demo
egg-demo.say_hello
{% endhighlight %}

成功输出！这说明安装正确。我们的一个 egg 包诞生了。 一般情况下，我们的源程序都放在 src 目录下，所以接下来将 egg_demo 文件夹移动到 src 里。但这样也要修改 setup.py 文件，修改 find_packages 函数中参数为 'src'，同时增加 package_dir 参数：

    packages=find_packages('src'),
    package_dir = {'':'src'}

这样告诉setuptools在src目录下找包，而不是原来默认的工程根目录。

Enjoy it!













