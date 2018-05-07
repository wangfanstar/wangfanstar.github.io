---
title: linux时间设计之一 -- 获取当前时间
tags: [linux,time]
category: 编程
date: 2018-05-07 20:19:03
---

##　linux时间设计之获取当前时间

买电脑主板的时候，会发现主板有个BIOS电池，里面保存着电脑的配置，还有一项就是时间，你设置好了之后就是电脑显示的当前时间。在Linux启动时，会有一个从CMOS中读取时间作为当前时间的操作，然后就再根据自己的时钟频率更新当前的时间。我们设置的时候会设置年月日时分秒，为了简化处理，linux把这些变量转换成一个变量jiffies，即从1970年到当前时间的所有毫秒数，之后硬件芯片每毫秒产生一次中断，在中断处理程序中jiffies就会加1，达到更新时间的效果。研究完Linux几个版本的将当前时间转换成秒数的时间设计后，分享一下其中的程序设计技巧和bug。

## linux 0.11版时间设计

linux 0.11版的时间设计很精炼，代码不到几十行，在`/kernel/mktime.c`里面实现了一个内核态计算当前开机距1970年1月1日0时的秒数，但估计闰年没考虑周全，闰年的定义网上看李永乐的视频讲得很生动。总结归纳起来就是能被4整除，不能被100整除或能被400整除的数。代码中只考虑了4年，100整数的2000年没有考虑进去，后面的几百年也没有考虑。

```c
#include <time.h>
/*
 * This isn't the library routine, it is only used in the kernel.
 * as such, we don't care about years<1970 etc, but assume everything
 * is ok. Similarly, TZ etc is happily ignored. We just do everything
 * as easily as possible. Let's find something public for the library
 * routines (although I think minix times is public).
 */
/*
 * PS. I hate whoever though up the year 1970 - couldn't they have gotten
 * a leap-year instead? I also hate Gregorius, pope or no. I'm grumpy.
 */
#define MINUTE 60
#define HOUR (60*MINUTE)
#define DAY (24*HOUR)
#define YEAR (365*DAY)

/* interestingly, we assume leap-years */
static int month[12] = {
	0,
	DAY*(31),
	DAY*(31+29),
	DAY*(31+29+31),
	DAY*(31+29+31+30),
	DAY*(31+29+31+30+31),
	DAY*(31+29+31+30+31+30),
	DAY*(31+29+31+30+31+30+31),
	DAY*(31+29+31+30+31+30+31+31),
	DAY*(31+29+31+30+31+30+31+31+30),
	DAY*(31+29+31+30+31+30+31+31+30+31),
	DAY*(31+29+31+30+31+30+31+31+30+31+30)
};

long kernel_mktime(struct tm * tm)
{
	long res;
	int year;

	year = tm->tm_year - 70;
/* magic offsets (y+1) needed to get leapyears right.*/
	res = YEAR*year + DAY*((year+1)/4);
	res += month[tm->tm_mon];
/* and (y+2) here. If it wasn't a leap-year, we have to adjust */
	if (tm->tm_mon>1 && ((year+2)%4))
		res -= DAY;
	res += DAY*(tm->tm_mday-1);
	res += HOUR*tm->tm_hour;
	res += MINUTE*tm->tm_min;
	res += tm->tm_sec;
	return res;
}
```



##　linu2.6.11版 时间设计 