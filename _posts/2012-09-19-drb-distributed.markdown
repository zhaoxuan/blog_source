---
layout: post
title: "转自 windy 分布式 DRb 的简介"
date: 2012-09-19 19:31
comments: true
categories: ruby
---
###分布式Ruby解决之道 - DR
>####分布式Ruby解决之道
>其实用Druby很久了，今天需要完成一个进程数据同步的机制，我需要的不是运行速度快，不是用 linux / mac 下的扩展，而是独立，快速开发效率，方便最简单的Ruby环境可运行，可以吗？ DRb(即分布式Ruby，下面都这样说它)是内置于Ruby标准库中的对象代理的实现。什么是对象代理，现在不明白不要紧，一会就知道了。

###解决什么样的问题？
>有的时候，我们需要提供远程的服务，比如提供远程API调用（如果你听过RPC，或WDSL），这样，我们可以很大程度上解耦各大模块，对外提供服务。
>
>还有的时候，我们需要在两个进程中通信，以获得互相的同步或资源。
>
>更有，我想实现实现某种透明的对象，让对象可以在不同的进程或主机上传递。
>
>这些，都可以通过 DRb 来实现。DRb 的相关文档非常少，但在想快速实现一个轻量级分布应用，依赖最少化时，使用它是非常方便的。我对分布式的研究不多，欢迎各位看了本文后能提出更多解决方案。

###使用方法
>依官方的例子为主，各位看官建议看的时候复制下试试。因为是分布式解决方案，肯定是 服务端 与 客户端 双方的代码。
{% highlight  ruby %}
require 'drb/drb'

# 监听的地址，你可以改为 0.0.0.0 来支持远程连接
URI="druby://localhost:8787"

class TimeServer
  def get_current_time
    return Time.now
  end
end

# 被代理的对象，客户端获取的到的对象就是它
FRONT_OBJECT=TimeServer.new

DRb.start_service(URI, FRONT_OBJECT)
# 
DRb.thread.join
{% endhighlight %}
{% highlight ruby %}
require 'drb/drb'

SERVER_URI="druby://localhost:8787"

# 这句是必要的，因为我们很快会用到回调与引用，一会说。
# 所以纯粹的客户端是不存在的。
DRb.start_service

timeserver = DRbObject.new_with_uri(SERVER_URI)
puts timeserver.get_current_time
{% endhighlight %}
>我必须要说的是，很符合我们的 C/S 模型，但是你有没有想过如果 `get_current_time` 返回一个远程对象，会发     生什么呢？ 接下来，就是我要讲的。

###远程对象代理
{% highlight ruby %}
require 'drb/drb'
URI="druby://localhost:8787"

class Logger

  # Logger 是被远程代理，客户端不会存在，所以用这句
  #注意这句话，如果要是自己来改写这个方法怎么做呢？下面会有我的一个例子
  include DRb::DRbUndumped

  def initialize(n, fname)
    @name = n
    @filename = fname
  end

  def log(message)
    File.open(@filename, "a") do |f|
      f.puts("#{Time.now}: #{@name}: #{message}")
    end
  end

end

class LoggerFactory

  def initialize(bdir)
    @basedir = bdir
    @loggers = {}
  end

  def get_logger(name)
    if !@loggers.has_key? name
      # 保证文件名是合法的
      fname = name.gsub(/[.\/]/, "_").untaint
      @loggers[name] = Logger.new(name, @basedir + "/" + fname)
    end
    return @loggers[name]
  end

end
# 在执行之前你要手动创建一下dlog
FRONT_OBJECT=LoggerFactory.new("dlog")

DRb.start_service(URI, FRONT_OBJECT)
DRb.thread.join
{% endhighlight %}

{% highlight ruby %}
require 'drb/drb'

SERVER_URI="druby://localhost:8787"

DRb.start_service

log_service=DRbObject.new_with_uri(SERVER_URI)

["loga", "logb", "logc"].each do |logname|

  logger=log_service.get_logger(logname)

  logger.log("Hello, world!")
  logger.log("Goodbye, world!")
  logger.log("=== EOT ===")

end
{% endhighlight %}

>吐嘈，执行完，你会发现日志被写在了服务端的 dlog/ 目录里，注意 DRb::DRbUndumped 在 Logger 对象的加载，这样的对象是无须传递给客户端的，这样，客户端代码里拿到的 loggger 对象是远程代理对象，所有该对象调用的方法实际上是在远程服务端执行的。我们称这种方法是按引用传递。
>
>那当然有一种传递叫，按值传递，什么情况是呢？显然，上面第一种方法即是，我们调用 get_current_time 是本地对象，再调用该对象的方法时，方法在本地执行。
>
>如此，便是 DRb 的基本使用方法了，应该说不难理解。你可以这样理解，都是对象，只不是有些对象是远程的，有些是本地的，远程的对象方法的执行是在远端，本地的方法是在本地。远程的对象是包含了 DRb::DRbUpdumped 的对象。不包含的都会转换为本地对象。
>
>那么，何为分布式的 Ruby，这明显是忽悠我们群众嘛？别急，我正要说，还记得一开始代码里注释的 start_service 了吧。所谓 服务端 可以随时获取 客户端 的远程对象，对吧？所以用 DRb 实现一个通信是非常简单的。为了有深入理解，我想需要将它的实现原理分析一下。

###如何实现的呢
>
>DRb 的本质是，一个通信底层，一个序列化方式，一个代理器，OK？你不用看都能知道是吧？因为我也会这样实现的。

####1.代理器
>
>method_missing 将一个对象的方法传递给另一个对象的神器，谓之代理，多像有关部门，不做事情，只是将事情移交给另一个有关部门。看看核心代码：
{% highlight ruby%}
def method_missing(msg_id, *a, &b)
  if DRb.here?(@uri)
    obj = DRb.to_obj(@ref)
    DRb.current_server.check_insecure_method(obj, msg_id)
    return obj.__send__(msg_id, *a, &b)
  end

  succ, result = self.class.with_friend(@uri) do
  DRbConn.open(@uri) do |conn|
    conn.send_message(self, msg_id, a, b)
  end
  #。。。处理异常
end
{% endhighlight %}
>obj显然是被代理的对象，上面除了缓存机制外，send_message 是 method_missing做的最重要的事，它引出来了下面的事情。

####2.通信底层
>
>DRb 的底层是一层透明的传输协议，通过它的接口，可以将数据（或命令）无压力收取，且看它的关键接口：
{% highlight ruby %}
def open(uri, config, first=true)
  @protocol.each do |prot|
    begin
      return prot.open(uri, config)
    rescue DRbBadScheme
    rescue DRbConnError
      raise($!)
    rescue
      raise(DRbConnError, "#{uri} - #{$!.inspect}")
    end
  end
  if first && (config[:auto_load] != false)
    auto_load(uri, config)
    return open(uri, config, false)
  end
  raise DRbBadURI, 'can\'t parse uri:' + uri
end

# drb/drb.rb:901 发送一个请求，通俗的说，调用一个方法
def send_request(ref, msg_id, arg, b)
  @msg.send_request(stream, ref, msg_id, arg, b)
end

# 在服务端，接受一个方法
def recv_request
  @msg.recv_request(stream)
end

# 服务端，发送一个结果
def send_reply(succ, result)
  @msg.send_reply(stream, succ, result)
end

# 客户端，接受一个结果
def recv_reply
 @msg.recv_reply(stream)
end
{% endhighlight %}
>继续吐嘈，默认 DRb 使用 DRbTCPSocket 来通信，你可以随时调整为 UnixSocket 或者 Http ，甚至 SSL。这个视你的需求而定，比如你要从公司用基于 Ruby 的方法，遥控你的家用电脑，建议你使用 SSL。
>
>抽象你的接口，是实现易于维护系统的关键，是吧。如何序列化是整个 DRb 的关键，而在 Ruby 中，这一切显得如此简单。
>
###序列化方法（与对象引用转换）
Marshal 神器用来序列化对象，默认直接使用即可。例如：
{% highlight ruby%}
class A
  def initialize(a)
    @a = a
  end
end
a = A.new(1)
b = Marshal.dump(a)
c = Marshal.load(b)
puts c.a  # ok, 输出 1
{% endhighlight %}
>它被引用在 DRb 中，做为 DRbMessage 的关键，传递对象使用。
>
>通过 Marshal 我们大概了解了一个序列化的例子，让我们来实现一个 Marshal example

>服务器端代码
{% highlight ruby %}
require 'drb/drb'

# 监听的地址，你可以改为 0.0.0.0 来支持远程连接
URI="druby://localhost:8787"

class A

  def initialize(a)
    @a = a
  end

  def a
    @a
  end
  
end

class TimeServer

  def get_current_time
    a = A.new('1')
    return Marshal.dump(a)
  end

end

FRONT_OBJECT=TimeServer.new

DRb.start_service(URI, FRONT_OBJECT)
# 
DRb.thread.join
{% endhighlight %}
>客户端代码
{% highlight ruby %}
require 'drb/drb'

SERVER_URI="druby://localhost:8787"

# 这句是必要的，因为我们很快会用到回调与引用，一会说。
# 所以纯粹的客户端是不存在的。
DRb.start_service

class A

  def initialize(a)
    @a = a
  end

  def a
    @a
  end
  
end

timeserver = DRbObject.new_with_uri(SERVER_URI)
a = timeserver.get_current_time
b = Marshal.load(a)
p b.a
{% endhighlight %}
>结果非常惊讶，序列化的对象又重新调用了方法，但是这个前提是保证反序列化时，可以创建这个对象，也就是复用的 class A 里面的代码

>于是，组合以上思路，DRb 就产生了，不过，我们还缺点什么没讲，作为安全的程序员，一定要看看。

>代理对象如果被发送了 instance_eval("rm -rf /") Ok，我们系统没了。。。

>所以，$SAFE = 1 是可以保障基本安全的， 然而，这还不够，更细的控制，应该由 Ruby 1.9.1 以后（应该是说我没深入研究过）开始的，我就不细说了，你如果有需求可以仔细看看。

>另一个问题是，分布式要求远程对象长期生效，那么你可以去研究下 DRb::TimerIdConv 进行生存期保存。

>最后一个问题，远程对象支持 block 调用吗？答案是，YES。 如何实现的呢？
{% highlight ruby%}
def perform_with_block
@obj.__send__(@msg_id, *@argv) do |*x|
 jump_error = nil
 begin
   block_value = block_yield(x) #本质是 block.call(*x)，只是特殊处理了 Array
 rescue LocalJumpError
   jump_error = $!
 end
 if jump_error
  case jump_error.reason
  when :break
    break(jump_error.exit_value)
  else
    raise jump_error
  end
end
block_value
end
{% endhighlight %}
>看的出来（再吐嘈），block是通过本地的调用后，将结果再传递给远程对象。详细可以继续看 drb/drb 里的 perform 实现。

>值得注意的是，如果一个对象没有 include DRb::DRbUndumped 被返回到客户端，则会抛出 DRbUnknownError 异常。这个很容易理解。另一个注意点是，一个类无法使用 Marshal.dump 时（例如打开了一个文件句柄），则需要想办法自己实现它，或者。。。或者你应该实现为远程代理类，对吧。

>好了，基本上都讲完了。代码里还有许多精华，例如 self.allcate 可以跳过 initialize 来创建一个类。

>看完后，你再想想开篇的需求是否可以轻松解决掉？实际上只需要几步：

>创建一个类，按一般方法编写它的方法。如果方法有返回自定义对象，根据是否远程代理加载 DRbUpdumped。

>加载 DRb ， 启动服务。

>客户端连接，获取代理对象，调用方法。

###与其他语言的解决方案的对比与区别

####JAVA的 RMI
>RMI 是JAVA的远程调用实现方法，这里有一篇不错的介绍：http://damies.iteye.com/blog/51778 。

>DRb 是分布式的，RMI是单向的 C/S。 DRb 不需要声明接口，直接使用。熟练后，可以极快速度完成一个通信和同步的应用。

####CORBA
>看这个：http://zh.wikipedia.org/wiki/CORBA ， 基本原理相同，不过 DRb 足够轻，足够快。

####WDSL
>利用xml的标准RPC调用。适合于静态语言。

>由于对其他的了解不深入，欢迎熟悉的看客们提出你的看法。

>其他需求

>在公司之前的工作时，需要将 JRuby 的对象代理到 Ruby 中，这样可以复用 gems 。

>需要远程API的方法调用另一个进程的所有方法。

>因为要代理所有本地不存在的对象，只使用 DRb 还不够。但基本思路很简单，利用一个模块的 const_missing 动态加载远程的对象，而远程对象在创建时均自动加载 DRbUpdumped 被远程代理。根据以上，我们可以写一个看似本地代码却可以轻易转到远程执行。

> 例如：
{% highlight ruby %}
require 'watir'
ie = Watir::IE.new
ie.goto("www.baidu.com") # 本地打开一个浏览器

# 加载为远程进程执行
ATU.require 'watir'
ie = ATU::Watir::IE.new
ie.goto("www.baidu.com") # 远程的进程打开一个浏览器
{% endhighlight %}

>有了它，几乎同一份代码可以同用两个用途。可以非常方便的以代码级的控制远程主机和对象，并且重用性很高。

>如何实现，可以自己想想，同时可以查看这里：[ruby_proxy的实现](https://github.com/windy/ruby_proxy)

>还有一篇 [slide](http://windy.github.com/ruby_proxy.html)

>推荐续读

>[一个让DRb真正分布式的rinda（Dave Thomas）](http://www.linux-mag.com/2002-09/ruby_05.html)

>[一些介绍的例子](http://segment7.net/projects/ruby/drb/introduction.html)

>转自 windy

>本文采用 署名 - 非商业 - 复制保留本授权 的方式进行发布。