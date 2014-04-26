---
layout: post
title: "How to get http request header"
date: 2014-04-26 15:13
comments: true
categories: linux
---
Recently, I need a function checking whether the file has downloaded completed, so there is a problem how to get size of a remote file without downloaded.

Using ruby, python or other language, I can't get the this from URL immediately.

So I have to come be to linux

    man wget
    man axel
    man curl
    ...

Yep, In the manual of curl, I get this

    curl -I, --head
    (HTTP/FTP/FILE) Fetch the HTTP-header only! HTTP-servers feature the command HEAD which  this  uses
    to  get  nothing  but the header of a document. When used on an FTP or FILE file, curl displays the
    file size and last modification time only.

Try, looking for the information You want.

Enjoy it.