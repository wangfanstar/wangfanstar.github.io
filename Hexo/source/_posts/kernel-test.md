---
title: linux 内核态函数的调试方法
tags: [kernel,linux]
category: 编程
date: 2018-05-04 21:06:17linuxli
---

## linux调试

通常我们在用户态下进行程序的编写，用一个main函数，gcc进行编译后直接在bash下执行编译生成的二进制文件即可看到程序运行结果。但内核态函数不是从main函数开始的，也无法直接在用户态运行，这时候我们一般采用module的方式来进行内核态函数的调试。

## module 的使用方法

以调试内核态函数printk为例。为调试pirntk函数的功能

```c
/*注意不能命名为Printk.c，因为Linux内部有这个文件，再添加同名的.ko文件会报错。*/
#include <linux/module.h>
#include <linux/init.h>

MODULE_LICENSE("Dual BSD/GPL");
static int hello_init(void)
{
    printk("hello world\n");
    return 0;
}
static void hello_exit(void)
{
    printk("hello bye\n");
    
}
module_init(hello_init);
module_exit(hello_exit);
```

makefile文件设置如下 

```makefile
obj-m = wf_printk.o
PWD := $(shell pwd)
all:
	make -C /lib/modules/$(shell uname -r)/build M=$(PWD) modules
clean:
	rm *.o *.ko  *.mod.c *.order *.symvers
```

测试命令如下

```bash
$ sudo chmod 777 wf_printk.c
$ sudo make
$ sudo insmod wf_printk.ko  #安装模块
$ dmesg -c  #查看效果

## 安装模块后现象如下
[ 8520.317794] 02:21:54.819662 timesync vgsvcTimeSyncWorker: Radical guest time change: 77 388 852 170 000ns (GuestNow=1 525 693 894 041 566 000 ns GuestLast=1 525 616 505 189 396 000 ns fSetTimeLastLoop=true )
[ 8845.422739] hello world

$ sudo rmmod wf_printk.ko  #卸载模块
$ dmesg -c  #查看效果

## 卸载模块后现象如下
[ 8520.317794] 02:21:54.819662 timesync vgsvcTimeSyncWorker: Radical guest time change: 77 388 852 170 000ns (GuestNow=1 525 693 894 041 566 000 ns GuestLast=1 525 616 505 189 396 000 ns fSetTimeLastLoop=true )
[ 8845.422739] hello world
[ 8925.981566] hello bye

```

