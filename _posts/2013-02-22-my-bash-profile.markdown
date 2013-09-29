---
layout: post
title: "我的 bash_profile"
date: 2013-02-22 15:56
comments: true
categories: linux
---
###习惯

对于我这样才接触 git 的人，总是有一个不好的习惯，就是不看 branch 就修改提交，因为原来使用 svn 根本没有分支这个概念。

###配置 .bash_profile
{% highlight bash %}
# show git branch
function git_branch {
    ref=$(git symbolic-ref HEAD 2> /dev/null) || return;
    echo "("${ref#refs/heads/}") ";
}

# show ruby version
function ruby_v(){
    ref=$(ruby -v|cut -c 1-10)||return;
    echo "("${ref}") ";
}

# PS1
export PS1="\[\033[01;33m\]\u@\h \[\033[1;32m\]\$(git_branch)$(ruby_v)\[\033[01;31m\]\W\$\[\033[00m\] "
{% endhighlight %}

两个方法一个是显示 git branch 分支，一个是显示 ruby version，因为使用 rvm 所以就写了这两个。