---
layout: post
title: "Truncating HTML while Preserving Structure"
date: 2014-02-12 16:40
comments: true
categories: ruby
---

Truncating HTML is a bit of a pain - in a recent effory to change the layout of my site, I updated the index page to show the last five posts - truncated, of course.

The problem with doing this is that the most common method of truncating a post is to remove all markup and truncate the remaining string. This would be fine, but for a site with a fair amount of colorized code examples (courtesy of [Pigments](http://pygments.org/ "Title")), it can look a little weird.

A better solution would be to truncate based on the number of characters, yet retain the markup, which is where [this cheeky snippet](https://github.com/MattHall/truncatehtml "Title") of code comes in.

Requirements

The truncator requires [Nokogiri](http://nokogiri.org/ "Title") to parse out the HTML string.

Usage

If you’re using Jekyll, add the html_filters.rb file to your _plugins directory - this will give you the helper truncatehtml as a [Liquid](http://liquidmarkup.org/ "Title") filter. In your views, you can use this function in the same way as you would use the normal truncate filter:

{% highlight html %}
page.content | truncatehtml: 500
{% endhighlight %}

How it works

Given a snippet of HTML, we use Nokogiri to parse it into a tree. This takes care of all of the messiness of dealing with HTML, and gives us back a tree of nodes representing the parsed snippet.

Now that we have the tree of nodes, we can traverse it depth-first. All text nodes are leaf nodes, so when we encounter one, we can count the length of the text.

Once we have all of the text we need, we continue traversing, but instead of counting text lengths, we delete all of the nodes we see after we’ve reached our maximum length.

After that, we have a tree of nodes that represent the truncated tree, and the can output the appropriate HTML.

[truncatehtml](http://codebeef.com/articles/truncating-html-while-preserving-structure/ "Title")