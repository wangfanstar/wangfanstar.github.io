---
title: linux上android 2.1 版本漏洞 setuid
tags: [linux]
category: 编程
date: 2018-05-22 21:50:36
---

# 一个android的提权漏洞

此文来自宋宝华讲课，提到在adb安卓的调试工具中，发现源码中有一个setuid(shell)的操作，但此操作没有判断返回值，因为工程师认为由root降权到shell权限的操作是肯定可以执行的。结果这个没有判断返回值的操作被黑客发现了。做成一键ROOT工具

# 用setuid来hack linux的原理

理论上从root降为shell肯定是可以的，但有一种情况，就是shell的pid已经用完，无法生成新的pid，这就导致setuid会返回失败，但因为没有判断返回值，导致setuid(shell)代码执行后没有降权，还是原来的root权限。黑客利用这一点，首先用fork程序将所有的pid用完，再调用setuid(shell)，这样就获取了root权限。