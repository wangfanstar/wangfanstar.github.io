---
title: linux 内核态函数的调试方法
tags: [kernel]
category: 编程
date: 2018-05-04 21:06:17linuxli
---

##　 linux调试

通常我们在用户态下进行程序的编写，用一个main函数，gcc进行编译后直接在bash下执行编译生成的二进制文件即可看到程序运行结果。但内核态函数不是从main函数开始的，也无法直接在用户态运行，这时候我们一般采用module的方式来进行内核态函数的调试。

## module 的使用方法

以调试内核态函数printk为例。为调试pirntk函数的功能