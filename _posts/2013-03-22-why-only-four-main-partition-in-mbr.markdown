---
layout: post
title: "为什么磁盘只能分四个主分区"
date: 2013-03-22 20:27
comments: true
categories: linux
---
不知道大家有没有考虑过，为什么一块磁盘一般只能分四个主分区，或者三个主分区加一个拓展分区。

为什么不能突破四个主分区的限制多分几个主分区呢？

当年 Bill Gates 说 “无论对谁来说,640K内存都足够了”。

硬盘的第一磁盘，第一磁道，第一扇区，是一个非常特殊的扇区，它是计算机的 Boot Sector。

MBR（Master Boot Record），DPT（Disk Partition Table）和 Boot Record ID 三部分就放在 Boot Sector 512 字节里面，看清楚是 512 字节，没有 k 字节。

MBR 又称为主引导记录，占用Boot Sector 的前 446 个字节（0~0x1BD），存放系统主引导程序（它负责从活动分区中装载并且运行系统引导程序），对于 window 7 系统，里面的代码会指向第一个主分区的 bootmgr，由 bootmgr 启动整个系统。

DPT 即主分区表占用 64 个字节（0x1BE~0x1FD），记录磁盘的基本分区信息。主分区表分为四个分区项，每项 16 个字节，分别记录每个主分区的信息（因此最多可以有四个主分区），而传统的 MBR 分区表只能识别磁盘前面的 2.2TB 左右的空间。

Boot Record ID即引导区标记占用两个字节（0x1FE~0x1FF），对于合法引导区，它等于 0xaa55，这是判别引导区是否合法的标志）