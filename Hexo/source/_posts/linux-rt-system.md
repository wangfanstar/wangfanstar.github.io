---
title: 宋宝华课程笔记2--linux实时系统
tags: [linux]
category: 宋宝华
date: 2018-05-28 04:12:12
---

## Real Time 实时系统的定义

linux不是硬实时的系统，是软实时。实时不是越快越好，是指可预期性。比如发射导弹时，必须保证在截止期限内发射出去，否则后果可能是灾难性的。硬实时强调在恶劣的情况下，从唤醒到任务真正执行之间的时间是可预期的，可以保证在预期的时间内执行到。Linux设计不保证是可预期的，因为Linux在中断，软中断，及spinlock的区间时是不可抢占的，这些区间内的执行时间是不可预期的。

抢占的时机很多，我们主要记住不可抢占的时机：1、中断，2、软中断，3、spinlock区间的进程。

如果在这三类任务时间内唤起了高优先级的任务，任务是不能抢占的，只有当CPU脱离了这三类区间后，高优先级的任务才可以进行抢占。

![linux_rt_4type_preempt](/img/linux_rt_4type_preempt.png)

![1527456325480](C:\Users\wangfan\AppData\Local\Temp\1527456325480.png)

如图所示，如果在一二三类区间唤醒了高优先级的进程，当前无法进行抢占，只有当CPU退回到第4类普通进程区间时才开始抢占。

linux的RT版本，https://wiki.linuxfoundation.org/realtime/start 这个项目是实现Linux的实时版本。主要原理是将Linux的中断线程化，即把一二三类区间都转换成四类区间。相应的linux 源码RT版本只针对特定的几个版本，使用方法是将代码中的RT补丁merge到对应的源码中，linux的RT可以做到100us量级的实时性能，（vxworks是几个us)，相应的吞吐性能也会下降。

安装linux的调度器的抢占模型选项

> 1、server版本 （不抢占）
>
> 2、desktop版本 （kernel 内不抢占）
>
> 3、no-latency desktop (kernel 内可抢占，手机桌面一般用此模型)
>
> 4、completely preemption   (kernel 内一二三类中断软中断都可抢占)

安装RT补丁后还是会有很多的代码BUG需要处理，比如内存管理方面，Linux的内存分配是LAZY式，当用到时才实际分配，安装实时补丁后有可能出现写内存时内存实际还未分配的情形。实时系统的另一个应用场景是安装2个系统：实时系统和Linux，比如单反之前用的都是实时操作系统，现在为了增加蓝牙，wifi等功能，用一个核执行实时系统，另一个核用Linux来实现蓝牙等功能。