---
title: linux下多线程，多进程程序设计
tags: [linux]
category: 编程
date: 2018-05-13 09:30:37
---

## 多进程设计

linux下调度程序对应的都task_struct，进程和线和没有区别，用户态中使用方法是fork，exec，wait三种流程。内核态中用的是kthread_create, kthread_run, wake_up_process的流程。查看进程pid及其状态的命令是top，根据进程，线程的不同显示的pid号含义也不一样，下面进行具体分析。

## 进程用户态父子进程设计

用户态进程调度是可抢占的，调度的对象是task_struct，要新建一个进程，可以调用fork函数来实现task_struct相关信息的复制，新进的进程称为子进程，在代码中和父进程是同一段代码。fork函数会返回两交，在代码区分父子进程部分的判定是通过fork的返回值，子进程在对应fork返回值等于0，在代码中可以通过判断if(0 == fork())后面添加子进程的代码处理。父进程的fork返回非0或错误码。父进程的代码在一般在子进程判断是否为0后的else中继续进行。

## 多进程样例

下面这段代码实现父子进程各自打印各自的Pid功能。

```c
#include <stdio.h>
#include <sys/types.h>
#include <sys/wait.h>
#include <unistd.h>

int main(int argc, char *argv[])
{
    pid_t stPid;
    if (0 == fork())
    {
        printf("this is child process pid: %d\n",getpid());
    }
    else
    {
        printf("this is parent process pid: %d\n",getpid());
    }
    return 0;
}

```

显示结果如下：

> this is parent process pid: 3607
>
> this is child process pid: 3608

## 设置进程执行顺序

在之前的代码中我们看到是父进程先打印出来的，linux在设计时不强制哪个进程先，两个是随机的，如果我们想让父进程在子进程之后执行，可以在父进程中使用等待函数wait，wait会等待子进程执行完成返回后再继续下一步，如下代码父进程会等等子进程后再执行，结果就可以按我们想要的顺序执行。

```c
#include <stdio.h>
#include <sys/types.h>
#include <sys/wait.h>
#include <unistd.h>

int main(int argc, char *argv[])
{
    pid_t stPid;
    if (0 == fork())
    {
        printf("this is child process pid: %d\n",getpid());
    }
    else
    {
        wait(NULL);
        printf("this is parent process pid: %d\n",getpid());
    }
    return 0;
}
```

使用了wait的多进程程序如下：

> this is child process pid: 3683
>
> this is parent process pid: 3682



## 异常注意事项

上述代码没有做异常处理fork是有可能不成功的，返回的pid<0要处理

```c
if（(stPid = fork()）< 0)
{
    ...
	errorhandle;
}
```

## pid查看命令

1、top 查看进程，top -H 查看线程

2、cat /proc/sys/kernel/threads-max 查看最大线程数

3、ps 查看线程信息

4、 ulimit -a 用来显示当前的各种用户进程限制

## TID PID TGID区别 

对于进程来说，取TID、PID、TGID都是一样的，取到的值都是相同的，但是对于线程来说

```
1. tid: 线程ID
2. pid: 线程所属进程的ID
3. tgid: 等同于pid，也就是线程组id，即线程组领头进程的id
```