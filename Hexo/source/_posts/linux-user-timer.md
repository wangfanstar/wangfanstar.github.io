---
title: linux用户态定时器的使用
tags: [linux]
category: 编程
date: 2018-05-21 21:04:04
---

#  用户态下三种定时器的linux使用方式

目前我们在用户态使用延时，定时的时候，通常最简单的方法是使用sleep系列的API，如下所示：

# sleep定时器系列

```c
#include <unistd.h>
unsigned int alarm(unsigned int seconds);

#include <unistd.h>
unsigned int sleep(unsigned int seconds);

#include <unistd.h>
int usleep(useconds_t usec);
    
#include <time.h>
int nanosleep(const struct timespec *req, struct timespec *rem);
                                            
```

glibc中，alarm通过调用间隔定时器itimer来实现，时间到后会产生SIGALRM信号，sleep调用alarm来实现 ，在alarm基础上注册了SIGALRM信号处理函数，Bsd的usleep通过select来实现，nanosleep直接调用内核的nanosleep实现，内核的nanosleep通过调用高精度定时器hrtimer实现。

# itimer间隔定时器系列

```c
#include <sys/time.h>

int getitimer(int which, struct itimerval *curr_value);
int setitimer(int which, const struct itimerval *new_value, struct itimerval *old_value);

结构定义：
struct itimerval {
    struct timeval it_interval; /* next value ：间隔时间*/
    struct timeval it_value;    /* current value：到期时间*/
};
struct timeval {
    time_t      tv_sec;         /* seconds */
    suseconds_t tv_usec;        /* microseconds */
};
```

> alarm sleep和间隔定时器itimer都是基于SIGALARM信号触发的，因此不能同时使用。为了能同时使用多个定时器，我们可以采用posix系统定时器

# POSIX定时器系列

posix定时器解决了itimer,sleep定时器的3个问题

> 1、同一进程同一时刻只能有一种同一类型（ITIMER_REAL，ITIMER_PROF，ITIMER_VIRT)的itimer，POSIX定时器可以在同一进程创建任意多个定时器
>
> 2、itimer定时器到期后只能通过SIGALRM，SIGVTALRM,SIGPROF信号通过进程，POSIX不仅可以通过信号，还可以自定义信号，启动一个线程来通过
>
> 3、itimer支持us级别，POSIX定时器支持ns级别

```c
int timer_create(clockid_t clock_id, struct sigevent *evp, timer_t *timerid)；
int timer_settime(timer_t timerid, int flags, const struct itimerspec *value, struct itimerspect *ovalue);
int timer_gettime(timer_t timerid,struct itimerspec *value);
int timer_getoverrun(timer_t timerid);
int timer_delete (timer_t timerid);

itimerspec定义与itimerspec类似，只是提供了ns级别的精度
struct itimerspec
{
    struct timespec it_interval;    // 时间间隔
    struct timespec it_value;       // 首次到期时间
};

struct timespec
{
    time_t  tv_sec    //Seconds.
    long    tv_nsec   //Nanoseconds.
};
```

