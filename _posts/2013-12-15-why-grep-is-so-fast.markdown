---
layout: post
title: "为什么 GNU grep 这么快"
date: 2013-12-15 17:39
comments: true
categories: shell
---
Linuxer 经常在 shell 输出的时候加一个 grep 在后面，把自己想要的东西输出，

{% highlight bash %}
ps -ef | grep java
{% endhighlight %}

然后 kill -9 (从前辈工作时，偷师学来的坏习惯)。

下面是一些简单的总结，关于为什么 GNU grep 如此之快

技巧1：GNU grep之所以快是因为它并不会去检查输入中的每一个字节。

技巧2：GNU grep之所以快是因为它对那些的确需要检查的每个字节都执行非常少的指令（操作）。

GNU grep使用了非常著名的 [Boyer-Moore](http://www.ruanyifeng.com/blog/2013/05/boyer-moore_string_search_algorithm.html "Title") 算法（译者注：BM算法，是一种非常高效的字符串搜索算法，一般情况下，比 [KMP](http://www.ruanyifeng.com/blog/2013/05/Knuth%E2%80%93Morris%E2%80%93Pratt_algorithm.html "Title") 算法快3-5倍，具体可查看这篇讲解非常详细的文章：grep之字符串搜索算法Boyer-Moore由浅入深（比 [KMP](http://www.ruanyifeng.com/blog/2013/05/Knuth%E2%80%93Morris%E2%80%93Pratt_algorithm.html "Title") 快3-5倍）），该算法首先从目标字符串的最后一个字符开始查找，并且使用一个查找表，它可以在发现一个不匹配字符之后，计算出可以跳过多少个输入字符并继续查找。
GNU grep还展开了 [Boyer-Moore](http://www.ruanyifeng.com/blog/2013/05/boyer-moore_string_search_algorithm.html "Title") 算法的内部循环，并建立了一个Boyer-Moore的delta表，这样它就不需要在每一个展开的步骤进行循环 退出判断了。这样的结果就是，在极限情况下（in the limit），GNU grep在需要检查的每一个输入字节上所执行的x86指令不会超过3条（并且还跳过了许多字节）。