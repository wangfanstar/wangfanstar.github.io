---
title: linux内核态和用户态9种通信方法之二--ioctl
tags: [linux,通信]
category: 编程
date: 2018-05-12 19:51:46
---

## ioctl：linux驱动的属性设置	

ioctl是linux提供用户态传递参数到内核态，用于更改设备相属性的一个系统调用。正常的设备有字符设备，块设备，网络设备，根据Unix哲学一切皆文件的思想，这些设备都是