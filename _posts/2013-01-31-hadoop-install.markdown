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
{% highlight bash %}
$ ssh localhost
{% endhighlight %}

如果不输入口令就无法用ssh登陆localhost，执行下面的命令：
{% highlight bash %}
$ ssh-keygen -t dsa -P '' -f ~/.ssh/id_dsa 
$ cat ~/.ssh/id_dsa.pub >> ~/.ssh/authorized_keys
{% endhighlight %}
 
####三、安装 hadoop
1、下载 hadoop 解压的任意目录

2、修改 hadoop_path/confi/hadoop_env.sh
{% highlight bash %}
export JAVA_HOME=path_to_java
export HADOOP_INSTALL=path_to_hadoop
export PATH=$PATH:$HADOOP_INSTALL/bin
{% endhighlight %}

####四、配置 hadoop

这里看你是用什么方式启动，我这里是使用 单机伪分布式，使用三个文件 core-site.xml, mapred-site.xml and hdfs-site.xml

####五、格式分布式文件

{% highlight bash %}
$ bin/hadoop namenode -format
{% endhighlight %}

####六、启动Hadoop守护进程：
{% highlight bash %}
$ bin/start-all.sh
{% endhighlight %}

Hadoop守护进程的日志写入到 ${HADOOP_LOG_DIR} 目录 (默认是 ${HADOOP_HOME}/logs).
浏览NameNode和JobTracker的网络接口，它们的地址默认为：

`NameNode - http://localhost:50070/`

`JobTracker - http://localhost:50030/`

将输入文件拷贝到分布式文件系统:
{% highlight bash %}
$ bin/hadoop fs -put conf input
{% endhighlight %}

运行发行版提供的示例程序：
{% highlight bash %}
$ bin/hadoop jar hadoop-*-examples.jar grep input output 'dfs[a-z.]+'
{% endhighlight %}

查看输出文件：
将输出文件从分布式文件系统拷贝到本地文件系统查看：
{% highlight bash %}
$ bin/hadoop fs -get output output 
$ cat output/*
{% endhighlight %}

或者
在分布式文件系统上查看输出文件：
{% highlight bash %}
$ bin/hadoop fs -cat output/*
{% endhighlight %}

完成全部操作后，停止守护进程：

{% highlight bash %}
$ bin/stop-all.sh
{% endhighlight %}
