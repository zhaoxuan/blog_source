---
layout: post
title: "提高 ruby script 的健壮性"
date: 2013-09-25 10:49
comments: true
categories: ruby
---

首先必须承认 ruby 脚本确实是一个非常好用的工具。

但是我们习惯了一个 script 解决所有的问题，比如

{% highlight ruby %}
# main.rb
require 'rubygems'

system 'rm -fr /tmp'
{% endhighlight %}

Soooooooo simple。

但是上面的文件，在其他项目里面被 `require 'main'` 会有什么后果?

代码就直接被执行了，后果可大可小，如果是 `system 'sudo rm -fr /'`。

从 python 借鉴了点东西，所有的代码还是放到 `def main(); end` 里面执行比较安全。

{% highlight ruby %}
# main.rb
def main
  do some code
end

if $0 == __FILE__
  main
end
{% endhighlight %}

在判断下当前程序是脚本还是被 `require` 的。







