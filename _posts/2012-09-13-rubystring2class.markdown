---
layout: post
title: "转 ruby  从一个字符串创建一个对象"
date: 2012-08-23 19:05
comments: true
categories: ruby
---
在 ruby 中，根据一个类名创建一个实例

在Ruby On Rails中文社区的的Ruby版上，karl问了一个有趣的问题，如何根据类名生成一个类的实例。
[链接](http://www.railscn.com/about2289.html)：

下面是一个Java语言的实现：

{% highlight java %}
    public static Object createObject(String className) 
        throws ClassNotFoundException, InstantiationException, IllegalAccessException {
        Class clazz = Class.forName(className);
        return clazz.newInstance();
    }
{% endhighlight %}

在Ruby中怎么做呢？karl自己给出了第一种做法：

{% highlight ruby %}
str = 'String'
eval "obj=#{str}.new"
{% endhighlight %}

这里利用了Ruby的动态语言的特性，在代码中执行代码。不过，照着尝试了一下，我有了一个疑问，如何使用生成的对象，也就是，我们如何利用这里的obj呢？

按照karl提供的方式，编写出这样的代码：

{% highlight ruby %}
str = 'String'
eval "obj = #{str}.new"
p obj
{% endhighlight %}


这里故意使用了p方法，因为对于一个空串，它可以打出""，而不是无声无息，便于查看。

这样的代码是无法运行的，异常如下：

{% highlight ruby %}
undefined local variable or method `obj' for main:Object (NameError)
{% endhighlight %}
然而，这段代码在irb中执行没有问题，这是irb和Ruby不兼容的一个地方。在《Programming Ruby》第二版中的第15章，谈及irb限制的时候，提到了这一点。

eval执行会有一个值，也就是它最后执行的语句的值，对应到这里就是obj的值。根据这个特性，改写得到了第二种做法，代码如下：

{% highlight ruby %}
str = 'Array'
obj = eval "obj = #{str}.new"
p obj
{% endhighlight %}

显然eval里的obj没有发挥什么作用，再修改一下：

{% highlight ruby %}
str = 'Array'
obj = eval "#{str}.new"
p obj
{% endhighlight %}

这样，我们就根据字符串创建出一个可用的对象。事实上，第一种方式也是可用的，不过，要obj在前面已经存在，只不过要稍做修改：

{% highlight ruby %}
obj = nil
eval "obj = String.new"
p obj
{% endhighlight %}

qiezi的回帖给出了一个我认为最漂亮的一种做法：

{% highlight ruby %}
obj = eval(str).new
{% endhighlight %}

同样是利用eval，不同的是，这里是直接执行类名，结果是得到一个类对象。调用类的new方法就创建实例。这段代码看上去类似于一个普通的函数调用，很简洁。

对应到前面的Java函数，Ruby的实现就可以这样写：

{% highlight ruby %}
def create_obj(class_name)
    eval(class_name).new
end
{% endhighlight %}

UPDATE

故事有了新的进展，njmzhang提供了另外的方法，按照他的说法，这几乎是一种标准用法：

{% highlight ruby %}
c = Object.const_get("Array")
s = c.new
{% endhighlight %}

这里利用到了一个事实，所有的类对象在Ruby中都是一个常量，所以，可以根据它的名字把这个常量提取出来。

除此之外，njmzhang还给出了另一种利用扩展实现的方法：

{% highlight ruby %}
c = Class.by_name("Array")
{% endhighlight %}

这里用到的[扩展](http://extensions.rubyforge.org/)在从方法的命名上来说，它比上一种方法更恰当一些。算上这两种实现，我最喜欢的仍然是qiezi给出的那种方法。
