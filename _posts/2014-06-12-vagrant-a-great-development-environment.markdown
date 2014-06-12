---
layout: post
title: "Vagrant, a great VM development environment"
date: 2014-06-12 18:23
comments: true
categories: ruby
---
Hadoop 生态圈的开发者不知道有没有这样的困惑，每次部署都会说：明明在我机子上面开发的没问题啊。

单机开发的，一到线上分布式的系统里面就是一堆 bug。

Vagrant 可以很好的解决这个问题。

###Vagrant安装

下载 <a href="https://www.virtualbox.org/">Virtual Box</a> 和 <a href="http://www.vagrantup.com/">Vagrant</a> 都不用说了。

###Vagrant配置

当我们安装好 VirtualBox 和 Vagrant 后，我们要开始考虑在VM上使用什么操作系统了，一个打包好的操作系统在 Vagrant 中称为 Box，即 Box 是一个打包好的操作系统环境，目前网络上什么都有，所以你不用自己去制作操作系统或者制作 <a href="http://www.vagrantbox.es/">Box:vagrantbox.es </a> 上面有大家熟知的大多数操作系统，你只需要下载就可以了，下载主要是为了安装的时候快速，当然Vagrant也支持在线安装。

Vagrant 会为每一个项目新建一个虚拟机，而 Box 就是虚拟机的原始镜像。

使用的时候可以进入项目目录

    vagrant init your_box.img

就可以新建虚拟环境，这个 Vagrantfile 文件就是虚拟机的配置文件，打开看看就知道 Vagrant 的强大，特别是针对多台机器的集群式的 VM 管理，真心佩服。

修改好了 Vagrantfile 文件，就能启动虚拟机了

    vagrant up

好了，少年，尽情安装的你 pkg 吧。

Enjoy it.