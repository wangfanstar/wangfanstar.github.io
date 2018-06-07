---
title: 宋宝华课程笔记3--linux 进程生命周期
tags: [linux]
category: 宋宝华
date: 2018-05-28 05:34:13
---

## linux的进程描述

进程是一个资源封装的单位，资源指占用的内存，文件系统，信号及处理方法。线程是调度执行的单元。一个进程区别与另一个进程的标记就是资源。linux操作系统是可以做到进程与进程之间的资源隔离。进程的描述就是资源的描述。PCB （PROCESS CONTROL BLOCK) 在不同操作系统中用于描述进程，在Linux的PCB就是用task_struct来描述。

> 1、mm 内存资源:  进程的内存
>
> 2、fs 文件系统资源1: 根路径和当前路径指针
>
> 3、 files 文件系统资源2: 进程打开的文件，文件描述符数组
>
> 4、sinal 信号资源：不同进程可以针对同一信号挂不同的处理方法 
>
> 5、pid 属性资源：描述进程的属性

![1527457308673](C:\Users\wangfan\AppData\Local\Temp\1527457308673.png)

### fork炸弹让linux死机

利用pid数量是有限的特性可以用于破坏Linux，比如linux下著名的fork炸弹就是利用不断产生进程把pid耗尽的方法来实现让linux死机。

```bash
#linux fork 炸弹
:(){:|:&};:
```

> :   函数名为冒号
>
> ()  函数参数定义
>
> {} 函数定义
>
> ：调用自己
>
> |：递归调用自己
>
> &  后台执行
>
> ;   函数结束
>
> ： 调用函数：

### pid数量限制导致安卓的一键root

总数Pid和用户的Pid都是有限的，可以通过ulimit命令来查看，max user processes 为对应的数量

```bash
wangfan@wangfan-VirtualBox:~$ ulimit -a
core file size          (blocks, -c) 0
data seg size           (kbytes, -d) unlimited
scheduling priority             (-e) 0
file size               (blocks, -f) unlimited
pending signals                 (-i) 15723
max locked memory       (kbytes, -l) 64
max memory size         (kbytes, -m) unlimited
open files                      (-n) 1024
pipe size            (512 bytes, -p) 8
POSIX message queues     (bytes, -q) 819200
real-time priority              (-r) 0
stack size              (kbytes, -s) 8192
cpu time               (seconds, -t) unlimited
max user processes              (-u) 15723
virtual memory          (kbytes, -v) unlimited
file locks                      (-x) unlimited
```

安卓的2.2.1之前的版本被发现一个漏洞，很容易就被一键root，安卓的调试软件adb刚开始时有root权限，之后adb调用api  setuid(shell) 把自己从root用户降为shell用户。谷歌的工程师在调用时没有检查setuid的返回值，即默认setuid总是可以成功。黑客们利用uid数量有限制的属性，将shell用户内的pid进程全部用完，这样调用setuid时是无法成功的，但因为没有检查返回值，导致adb调用setuid(shell) 后没有降权成功，还是有root权限。这就是Android著名的提权漏洞：rageagainstthecage。2.2之后的安卓版本修复了此漏洞，方法是检查setuid的返回值。

## linux进程task_struct的三种数据结构

linux的进程用了三种数据结构来描述task_struct

> 1、链表：
>
> 2、树：  描述进程和子进程之间的关系
>
> 3、哈希表：

### 