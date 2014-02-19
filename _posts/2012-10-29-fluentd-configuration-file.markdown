---
layout: post
title: "Fluentd 配置文件"
date: 2012-10-29 07:56
comments: true
categories: log
---
<a href="http://docs.fluentd.org/articles/config-file" target="_blank">Fluentd config file</a>
Overview
The configuration file allows the user to control the input and output behavior of Fluentd by (1) selecting input and output plugins and (2) specifying the plugin parameters. The file is required for Fluentd to operate properly.

####List of Directives

The configuration file consists of the following directives:
> 1.source directives determine the input sources.
>
> 2.match directives determine the output destinations.
>
> 3.include directives include other files.

####Source Directive
Fluentd’s input sources are enabled by selecting and configuring the desired input plugins using source directives. Fluentd’s standard input plugins include http and forward.
{% highlight bash %}
<source>
  type http
  port 8888
</source>
{% endhighlight %}

####Match Directive
Fluentd’s output destinations are enabled by selecting and configuring the desired output plugins using match directives. Fluentd’s standard output plugins include file and forward.

{% highlight bash %}
<match debug.**>
  type stdout
</match>
{% endhighlight %}

Each match directive must include a match pattern and a type parameter. Match patterns are used to filter the events. Only events with tag matching the pattern will be sent to the output destination. The type parameter specifies the output plugin to use.

####Match Pattern: how you control the event flow inside fluentd

The following match patterns can be used for the <match> tag.

    * matches a single tag part.
    For example, the pattern a.* matches a.b, but does not match a or a.b.c

    ** matches zero or more tag parts.
    For example, the pattern a.** matches a, a.b and a.b.c

    {X,Y,Z} matches X, Y, or Z, where X, Y, and Z are match patterns.
    For example, the pattern {a,b} matches a and b, but does not match c
    This can be used in combination with the * or ** patterns. Examples include a.{b,c}.* and a.{b,c.**}







