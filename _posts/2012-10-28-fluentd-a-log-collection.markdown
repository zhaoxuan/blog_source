---
layout: post
title: "使用 Fluentd 构建一个实时日志收集系统"
date: 2012-10-28 07:29
comments: true
categories: fluentd ruby log
---

Fluentd 是一个日志收集系统，它的特点在于其各部分均是可定制化的，你可以通过简单的配置，将日志收集到不同的地方。

####安装
Fluentd 目前已经提供了多个版本的安装包，包括 Debian 系列的 apt 安装包，Red Hat 系列的 yum 安装包，也包括 Ruby 的 gem 安装包。
{% highlight bash %}
$ gem install fluentd --no-ri --no-rdoc
{% endhighlight %}

####运行
{% highlight bash %}
$ fluentd --setup ./fluent
$ fluentd -c ./fluent/fluent.conf -vv &
$ echo '{"json":"message"}' | fluent-cat debug.test
{% endhighlight %}