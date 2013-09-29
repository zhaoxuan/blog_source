---
layout: post
title: "hadoop install"
date: 2013-01-31 13:43
comments: true
categories: feeling
---
####一 软件安装环境
>1、 OS：ubuntu-12.10

>2、 JDK：jdk1.7.0_07

>3、 Hadoop：hadoop-1.0.4

>4、 ssh ssh-server

安装的方法就自己去 google 搜索吧，这里就不啰嗦了。


####二 配置 ssh

现在确认能否不输入口令就用ssh登录localhost:

{% highlight ruby %}
$ ssh localhost
如果不输入口令就无法用ssh登陆localhost，执行下面的命令：
$ ssh-keygen -t dsa -P '' -f ~/.ssh/id_dsa 
$ cat ~/.ssh/id_dsa.pub >> ~/.ssh/authorized_keys
{% endhighlight %}



$ ssh localhost
如果不输入口令就无法用ssh登陆localhost，执行下面的命令：
$ ssh-keygen -t dsa -P '' -f ~/.ssh/id_dsa 
$ cat ~/.ssh/id_dsa.pub >> ~/.ssh/authorized_keys

 
三、安装 hadoop
1、下载 hadoop 解压的任意目录
2、修改 hadoop_path/confi/hadoop_env.sh
     export JAVA_HOME=/usr/java/jdk1.7.0_07