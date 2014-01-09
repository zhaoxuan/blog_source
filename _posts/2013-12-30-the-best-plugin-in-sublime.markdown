---
layout: post
title: "The Best Plugin in Sublime Text"
date: 2013-12-30 23:20
comments: true
categories: sublime
---
这是我用过 Sublime 最好用的插件，没有之一。

CodeIntel 可以使一个简单的文本编辑器变成一个强大的 IDE。

###Installing
With the Package Control plugin: The easiest way to install SublimeCodeIntel is through Package Control, which can be found at this site: <a href="http://wbond.net/sublime_packages/package_control" target="_blank">Package Control</a>

Once you install Package Control, restart Sublime Text and bring up the Command Palette (Command+Shift+P on OS X, Control+Shift+P on Linux/Windows). Select "Package Control: Install Package", wait while Package Control fetches the latest package list, then select SublimeCodeIntel when the list appears. The advantage of using this method is that Package Control will automatically keep SublimeCodeIntel up to date with the latest version.

###Configuring
For adding additional library paths (django and extra libs paths for Python or extra paths to look for .js files for JavaScript for example), either add those paths as folders to your project, or create an optional codeintel configuration file in your home or in your project's root.

Configuration files (~/.codeintel/config or project_root/.codeintel/config). All configurations are optional.

This is my configuration file:
{% highlight bash %}
{
  "Python": {
    "python": '/usr/bin/python',
    "pythonExtraPaths": ["/Library/Python/2.7/site-packages"]
  },

  "Ruby": {
    "ruby": "~/.rvm/bin/default_ruby",
    "rubyExtraPaths":["~/.rvm/gems/ruby-1.9.3-p448@r9/bin"]
  }
}
{% endhighlight %}