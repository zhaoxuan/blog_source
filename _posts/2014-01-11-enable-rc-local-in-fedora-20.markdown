---
layout: post
title: "Enable rc-local in Fedora 20"
date: 2014-01-11 09:59
comments: true
categories: linux fedora systemctl
---
From http://joshua14.homelinux.org/blog/?p=1377

As root:

{% highlight bash %}
touch /etc/rc.local
chmod +x /etc/rc.local
{% endhighlight %}

Next, open the file for editing:-

vi /etc/rc.local
As this is basically a bash script itself, you need to include the bash interpreter as the first line of the file.

{% highlight bash %}
#!/bin/bash
{% endhighlight %}

You can now add whatever scripts or commands you like here – they will be run after everything else at your specific run-level has been started. In order for systemd to recognise and use this file, the systemd rc-local.service must be enabled.

{% highlight bash %}
systemctl enable rc-local.service
{% endhighlight %}

You can check the status of this service with:-

{% highlight bash %}
systemctl status rc-local.service
{% endhighlight %}