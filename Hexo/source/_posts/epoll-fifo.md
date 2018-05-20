---
title: 特殊条件下对epoll的改进fifo方案
tags: [linux]
category: 编程
date: 2018-05-19 06:28:29
---

# 多路复用

用epoll代替select，效率上有了很大提高 。在单线程中，epoll相当于堵塞在当前节点等待事件发生，

