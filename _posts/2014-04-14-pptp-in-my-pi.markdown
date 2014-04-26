---
layout: post
title: "Run all traffic through an PPTP VPN on my Raspberry Pi"
date: 2014-04-14 09:01
comments: true
categories: pi
---
#### VPN Client

Some time, I have to run all traffic through a VPN, because this is China.

Welcome to China.

It is so easy to set VPN in Mac or Gnome environment desk, but I didn't have used it in command line before.

To start, You need install pptp client in Pi, this can be achieved by:

    sudo pacman -S pptp-client

Next, Create a pptp account:

    pptpsetup --create my_vpn --server vpn.example.com --username john --password foo --encrypt

Then, You can find the configured file in /etc/ppp/peers

To make sure that everythine is configured properly, as root execute

    # pon <TUNNEL> debug dump logfd 2 nodetach

If everything is configured correctly, the pon command should not terminate. Once you are satisfied that has connected successfully, you can terminate this command.

To connect to your VPN normally, simply execute:

    # pon <TUNNEL>

#### ROUTING

Once you have connected your VPN, you should be able to interact with anything available on th VPN. To access anything on the remote server, you should add a new route in your routing table.

Route all traffic through your vpn.

    sudo route del defalut dev eth0
    sudo route del defalut gw 10.1.0.1 dev ppp0

Check whether it has worked

    ping www.youtube.com

Enjoy it.