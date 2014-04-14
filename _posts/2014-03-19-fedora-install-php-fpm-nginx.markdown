---
layout: post
title: "Fedora install php-fpm and nginx"
date: 2014-03-19 21:24
comments: true
categories: linux
---
### Install nginx and php-fpm

{% highlight bash %}
sudo yum install nginx php-fpm
{% endhighlight %}

### Test nginx

{% highlight bash %}
sudo nginx
{% endhighlight %}

Type in your web server's IP address or hostname into a browser (e.g. http://127.0.0.1), and you should see the following page: nginx

All right

### My nginx conf

{% highlight bash %}
sudo vim /etc/nginx/nginx.conf
{% endhighlight %}

Change user and group to john:john

    user john john

Set root path to my home path

    root         /home/john/www;

Set php-fpm script

    location ~ \.php$ {
        root /home/john/www;
        fastcgi_pass 127.0.0.1:9000;
        fastcgi_index index.php;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        include fastcgi_params;
    }

### My php-fpm conf

{% highlight bash %}
sudo vim /etc/php-fpm.d/www.conf
{% endhighlight %}

Nginx choosed to be able to access my dir ad httpd

    user = john
    group = john
