---
layout: post
title: "Run all traffic through an PPTP VPN"
date: 2014-04-14 09:01
comments: true
categories: pi
---
Some time, I have to run all traffic through a VPN, because this is China.

Welcome to China.

It is so easy to set VPN in Mac or Gnome environment desk, but I didn't use it in command line.

To start, You need install pptp client in Pi, this can be achieved by:

    sudo pacman -S pptp-client

Next, Create a pptp account:

    pptpsetup