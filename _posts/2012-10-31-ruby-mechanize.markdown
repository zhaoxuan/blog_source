---
layout: post
title: "使用 mechanize 模拟浏览器提交数据"
date: 2012-10-31 22:06
comments: true
categories: ruby
---

对于要抓取网页的操作，nokogiri 绝对是一个非常牛逼的 gem。

但是如果这个页面涉及到登陆，涉及到跳转的时候 nokogiri 就有点无能为力了。

这里推荐使用 mechanize，类似的还有 faraday。

如果要涉及到 cookie 操作的话，推荐使用 mechanize，这样跳转起来方便。

如果要是抓 API 的话，就用 faraday。

1 install mechanize
{% highlight ruby %}
gem install mechanize
{% endhighlight%}

2 mechanize object
{% highlight ruby %}
agent = WWW::Mechanize.new
{% endhighlight %}

3 vitual user_agent
{% highlight ruby %}
agent.user_agent_alias = 'Windows IE 7'
{% endhighlight %}

``` ruby user_agent https://github.com/sparklemotion/mechanize/blob/master/lib/mechanize.rb#L115 detail
AGENT_ALIASES = {
    'Mechanize' => "Mechanize/#{VERSION} Ruby/#{ruby_version} (http://github.com/sparklemotion/mechanize/)",
    'Linux Firefox' => 'Mozilla/5.0 (X11; U; Linux i686; en-US; rv:1.9.2.1) Gecko/20100122 firefox/3.6.1',
    'Linux Konqueror' => 'Mozilla/5.0 (compatible; Konqueror/3; Linux)',
    'Linux Mozilla' => 'Mozilla/5.0 (X11; U; Linux i686; en-US; rv:1.4) Gecko/20030624',
    'Mac Firefox' => 'Mozilla/5.0 (Macintosh; U; Intel Mac OS X 10.6; en-US; rv:1.9.2) Gecko/20100115 Firefox/3.6',
    'Mac Mozilla' => 'Mozilla/5.0 (Macintosh; U; PPC Mac OS X Mach-O; en-US; rv:1.4a) Gecko/20030401',
    'Mac Safari 4' => 'Mozilla/5.0 (Macintosh; U; Intel Mac OS X 10_6_2; de-at) AppleWebKit/531.21.8 (KHTML, like Gecko) Version/4.0.4 Safari/531.21.10',
    'Mac Safari' => 'Mozilla/5.0 (Macintosh; Intel Mac OS X 10_7_2) AppleWebKit/534.51.22 (KHTML, like Gecko) Version/5.1.1 Safari/534.51.22',
    'Windows IE 6' => 'Mozilla/4.0 (compatible; MSIE 6.0; Windows NT 5.1)',
    'Windows IE 7' => 'Mozilla/4.0 (compatible; MSIE 7.0; Windows NT 5.1; .NET CLR 1.1.4322; .NET CLR 2.0.50727)',
    'Windows IE 8' => 'Mozilla/5.0 (compatible; MSIE 8.0; Windows NT 5.1; Trident/4.0; .NET CLR 1.1.4322; .NET CLR 2.0.50727)',
    'Windows IE 9' => 'Mozilla/5.0 (compatible; MSIE 9.0; Windows NT 6.1; Trident/5.0)',
    'Windows Mozilla' => 'Mozilla/5.0 (Windows; U; Windows NT 5.0; en-US; rv:1.4b) Gecko/20030516 Mozilla Firebird/0.6',
    'iPhone' => 'Mozilla/5.0 (iPhone; U; CPU like Mac OS X; en) AppleWebKit/420+ (KHTML, like Gecko) Version/3.0 Mobile/1C28 Safari/419.3',
  }
```

4 get page
``` ruby get_page
agent.get('https://my.comscore.com/login/') 
```

5 set time out
``` ruby set_time_out 
agent.open_timeout = 10 
```

6 read form tag
``` ruby read_form_tag
page.form('loginform')
```

7 fill in form
``` ruby fill
login_form.userid = 'username'
login_form.passwd = 'password'
```

8 submit form
``` ruby submit
agent.submit(login_form)
```

这样就能完成对 comscore 这个网站的登陆了，非常简单好用。


