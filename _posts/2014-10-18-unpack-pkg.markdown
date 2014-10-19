---
layout: post
title: "分析 pkg 文件"
date: 2014-10-18 16:44
comments: true
categories: mac
---
```pkg``` 文件是 OS X 上特有的安装文件，我感觉和 RedHat 上面 RPM Package 差不多，里面有安装程序，有解压出来的二进制文件和库文件。

假设当前有一个 pkg 文件 "filename.pkg"，先使用以下命令解开 pkg：

    $ xar -xf filename.pkg

如果是 Linux 系统，可以 Google 去下载安装 [xar](http://code.google.com/p/xar/ "Title")。

解压后可以看见如下文件：

    Distribution
    Resources
    filename1.pkg
    filename2.pkg

```Distribution``` 是安装程序的 GUI 代码，```Resources``` 这个目录还不了解。

剩下若干 ```*.pkg``` 文件夹，就是我们要安装的程序的文件，随便进入一个 ```.pkg``` 目录，可以看见这样的 4 个文件：

    Bom
    PackageInfo
    Payload
    Scripts

其中的 ```Payload``` 保存了我们需要的文件，需要使用以下命令对其进行处理：

    $ mv Payload Payload.gz
    $ gunzip Payload.gz
    $ cat Payload | cpio -i

解压 ```Payload``` 以后我们就能看到和我们 OS X 类似的目录结构：

    $ ls
    Applications
    Library
    System
    ...

如果有 .app 就在 ```Applications``` 下面，如果有库文件就在 ```Library``` 下面，所以以后如果 pkg 安装的程序没有删除工具的话，可以用这样的方式删除。












