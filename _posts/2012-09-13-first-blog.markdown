---
layout: post
title: "octopress 安装"
date: 2012-07-13 15:12
comments: true
categories: ruby
---
##octopress 安装
安装与发布

Octopress如果你不了解Ruby以及Git，不建议急着上手安装Octopress。安装步骤你可以参考官网文档，也可以按照以下步骤来安装。
a. Octopress需要基于ruby 1.9.2版本环境，通过RVM切换

    rvm install 1.9.2 && rvm use 1.9.2

b. 通过git clone获取octopress源代码

    git clone git://github.com/imathis/octopress.git octopress

c. 进入octopress文件夹，将Gemfile的source改用淘宝镜像

    source "http://ruby.taobao.org"

d. 运行命令进行安装

    bundle install
    rake install

e. 建立github pages repo，假设你的用户名是username，则建立repo名为username.github.com/repo。具体可参见github pages。

f. 通过以下命令，自动建立github pages所需的配置

    rake setup_github_pages[git@github.com:username/username.github.com.git]

当提示输入github url时，请输入git@github.com:username/username.github.com.git，当然需要替换username。
该命令将自动建立两个branch，分别是master和source。master用来发布octopress生成的静态内容，在deploy文件夹里的静态文件。source用来发布除了deploy之外的所有文件，既octopress程序本身。

g. 通过_config.yml配置octopress博客信息，请注意冒号之后需要空格，具体详见Octopress Configuring。
h. 生成以及预览功能

    rake generate # generate your blog static pages content according to your input. 
    rake preview # start a web server on "http://localhost:4000", you can preview your blog content.

第一条命令将生成静态页面到_deploy文件夹；第二条命令通过http://localhost:4000来预览生成的页面，这时候还没有写文章，先看看是否config信息是否正确。

i. 设置 github ssh 验证

    cd ~/.ssh
    ssh-keygen -t rsa -C "your_email@youremail.com"
    # Creates a new ssh key using the provided email

j. 增加 ssh_key 到 github 里面

>1 Go to your [Account Settings](https://github.com/settings/profile)
>
>2 Click "[SSH Keys](https://github.com/settings/ssh)" in the left sidebar
>
>3 Click "Add SSH key"
>
>4 Paste your key into the "Key" field
>
>5 Click "Add key"
>
>6 Confirm the action by entering your GitHub password

k. 测试 github ssh_key
    ssh -T git@github.com
    Hi zhaoxuan! You've successfully authenticated, but GitHub does not provide shell access.

l. 发布生成的静态内容到master branch，在 deploy 之前，你还不能直接提交代码到 github 里面，因为 ssh 还要验证身份，所以必须要在账户里面设置 ssh

    rake deploy # push your static pages content to your github pages repo ("master" branch)

m. OK 打开你的 blog，好好编辑你的 blog 吧
    http://username.github.com/repo