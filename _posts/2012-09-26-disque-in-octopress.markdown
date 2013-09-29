---
layout: post
title: "在 octopress 使用 disqus 的评论功能"
date: 2012-09-26 14:11
comments: true
categories: blog
---
看见别人一直都是有 comments 在每一篇 blog 下面的，但是为什么我的没有呢？马上加入一个。

###1.配置 _config.yml 文件
    disqus_short_name: your_shortname
    disqus_show_comment_count: true

###2.申请一个 DISQUS 帐号
[DISQUS](http://www.disqus.com)

>在 DISQUS 里面加入一个 blog 页面
{{ '/images/yujinxiang.jpeg' | asset_url | img_tag }}
![Alt text](/images/yujinxiang.jpeg)

###3.复制代码
把最下面的脚本复制到 source/_includes/post/disqus_thread.html 里面去。ok 刷新页面把，你能看到别人的评论了。