---
layout: post
title: "Sqlite 也能大数据"
date: 2013-02-11 15:18
comments: true
categories: sqlite
---
### 简介

SQLite 是一个开源的嵌入式关系数据库，实现自包容、零配置、支持事务的SQL数据库引擎。 其特点是高度便携、使用方便、结构紧凑、高效、可靠。 与其他数据库管理系统不同，SQLite 的安装和运行非常简单，在大多数情况下 - 只要确保SQLite的二进制文件存在即可开始创建、连接和使用数据库。

但是今天，我并不是要在嵌入式开发中用它，因为他的简单，小巧，我反而要在大数据中使用它。

### 使用

在 ruby 中使用 SQLite 是非常简单的：

{% highlight ruby %}
require "sqlite3"

db = SQLite3::Database.new(db_path)
{% endhighlight %}

还有比这个更简单的吗？

### 背景

ID3 算法里面决策树的分裂是父子关系的，子节点的数据都是在父节点的条件下分裂的。

|Outlook|Temperature|Humidity|Windy|Play?|
|---|---|---|---|---|
|sunny|hot|high|false|no|
|sunny|hot|high|true|no|
|overcast|hot|high|false|yes|
|rain|mild|high|false|yes|
|rain|cool|normal|false|yes|
|rain|cool|normal|true|no|
|overcast|cool|normal|true|yes|
|sunny|mild|high|false|no|
|sunny|cool|normal|false|yes|
|rain|mild|normal|false|yes|
|sunny|mild|normal|true|yes|
|overcast|mild|high|true|yes|
|overcast|hot|normal|false|yes|
|rain|mild|high|true|no|

比如，通过 Gain 计算，使用 Attribute = Outlook 来分裂决策树，Values = [sunny, overcast, rain]

树就会变成这个样子 ![decision tree]({{ site.url }}/images/post/decision_tree.jpg)

这样数据就会根据 $$value \in [sunny, overcast, rain]$$ 分成三个集合。

然后再重复执行上面的步骤，直到把树的枝叶都分裂完成。

### 方案

在技术讨论的时候，这时候输入的数据不是上面的例子这几条，而是上亿的 impressions。

这样的数据在一台 Mysql 里面查询已经明显感觉到有点吃力。还好我们有 [Greenplum](http://www.greenplumdba.com/ "Title") 算是没有浪费什么事就解决的数据的存储问题。

但是 [Greenplum](http://www.greenplumdba.com/ "Title") 分布式数据库的查询是非常慢的，毕竟所有数据要在一台机器上做聚合，这里就不说明原因了。

所以整个决策树在生成的时候都要运行 2-3 天的时间。最后讨论了俩个方案：

    1. 分表，根据 attribute 条件来分表，每一个节点和子节点都是用父节点的字表来运算。
    2. 用缓存，我当时强烈推荐使用缓存，把节点下面的字表缓存在内存里面，这样就能一次查询和子查询就不用访问 greenplum 了。

两种方案其实做法一样都是要大表分小表，只是实现不一样，一个主张在数据库里面创建小表，一个使用缓存来在客户端缓存数据。

最后还是用了 SQLite 来做一个缓存的方式来完成了工作。

### 实现

SQLite 其实可以在内存里面

{% highlight ruby %}
cache = SQLite3::Database.new ":memory:"
{% endhighlight %}

依据决策树的特点，孩子节点的数据肯定是父节点的条件顾虑出来的。

{% highlight ruby %}
if in_cache?(opt)
  puts "\e[31;1m==> exec sql in sqlite :\e[0m #{sql}"
  return cache_table_exec(sql)
else
  puts "\e[31;1m==> exec sql in greenplum :\e[0m #{sql}"
  return gp_exec(sql)
end
{% endhighlight %}

in_cache? 其实就是一个判断当前节点的数据是不是上次缓存的数据的子集，如果是就用 sqlite 里面的数据查询，不是就连接 gp 数据查询。

这样既保证决策树用 sql 来查询数据的功能不变，还能加快速度。

时间从原来的 2-3 天，现在可以在 1 个小时内完成。


