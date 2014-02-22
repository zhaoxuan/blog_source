---
layout: post
title: "使用 ruby 解析纯真 IP 库"
date: 2014-02-19 09:36
comments: true
categories: ruby
---
### QQ 纯真 IP 库

通过 IP 来定位 Region 已经是在 RTB 里面非常常用的手段，但是现在客户需要更加精准的定位人群的位置。

所以就想到了使用纯真 IP 库，来更加精准的定位。

### QQWry.dat 结构

QQWry.dat文件在结构上分为3块：文件头，记录区，索引区。

一般我们要查找IP时，先在索引区查找记录偏移，然后再到记录区读出信息。由于记录区的记录是不定长的，所以直接在记录区中搜索是不可能的。由于记录数比较多，如果我们遍历索引区也会是有点慢的，一般来说，我们可以用二分查找法搜索索引区，其速度比遍历索引区快若干数量级。图1是QQWry.dat的文件结构图。

![图1]({{ site.url }}/images/post/qqwry_dat_pic1.gif)

QQWry.dat的文件头只有8个字节，其结构非常简单，首四个字节是第一条索引的绝对偏移，后四个字节是最后一条索引的绝对偏移。

{% highlight ruby %}
@file = File.open('qqwry.dat',"r")
@index_first, @index_last  = @file.read(8).unpack('L2')
{% endhighlight %}

有了索引区的绝对偏移，通过 IP 获得相对偏移，然后在索引区我们就能顺着索引指向的地方去读取内容了。

{% highlight ruby %}
ip_long = ip.to_long

offset_min,offset_max = @index_first,@index_last

while offset_min <= offset_max
  offset_mid = offset_min + (offset_max - offset_min) / 14*7
  mid = read_long(offset_mid)

  if ip_long < mid
      offset_max = offset_mid - 7
  elsif ip_long == mid
      return read_offset(offset_mid+4)
  else
      offset_min = offset_mid + 7
  end
end

return read_offset(offset_max+4)
{% endhighlight %}

通过 read_offset 返回的内容，我们就知道数据存放的地址了，就能直接读取里面的内容了。

这里还有一点需要注意下，qqwry.dat 是用 gb2312 方式编码的，还需处理一下。

详细的代码可以在 [QQ 纯真 IP 库](https://github.com/zhaoxuan/qqwry "Title") 里面找到。

其实早就有了 Gem 可以实现这个功能，为了学习才自己研究一下。

{% highlight ruby %}
require 'qqwry'

db = QQWry::Database.new('QQWry.Dat')
r = db.query('8.8.8.8')
puts "#{r.country} #{r.area}"
{% endhighlight %}
