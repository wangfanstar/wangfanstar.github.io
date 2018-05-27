---
title: 一个安全的字符串拼接功能设计
tags: [linux,c]
category: 编程
date: 2018-05-23 18:07:59
---

# 原有的字符串拼接函数缺陷

sprintf,snprintf,scnprintf三个函数分别代表了Linux中对字符串进行处理的三个阶段。最终的scnprintf版本可以安全地进行字符串拼接，但还是有些麻烦，比如要计算字符长度，拼接时要先算出之前字符串的长度，如下所示，如果要拼接字符串a，和字符串b，用scnprintf实现就是先申请一个长度为c的空间，将a和b写入c空间中，如下所示。

# ibuf设计要解决的问题

> 1. 拼接时不用计算前面字符串的长度
> 2. 超过长度时自动截断，能安全地实现超规格的处理
> 3. 使用方便。

# scnprintf相关文件

scnprintf搜索在Linux发现只在kernel.h中，用户态中没有相关的实现。在网站中搜索如下所示：

```c
include/linux/kernel.h, line 330 (as a prototype)
static inline int scnprintf(char * buf, size_t size, const char * fmt, ...)
{
	va_list args;
	ssize_t ssize = size;
	int i;

	va_start(args, fmt);
	i = vsnprintf(buf, size, fmt, args);
	va_end(args);

	return (i >= ssize) ? (ssize - 1) : i;
}
```

# 内核态测试文件

bash是在用户态的，要测试内核态的程序需要用户态与内核态通信，见内核态与用户态的9种通信方式 ，下面用最简单的2种驱动模块安装和proc文件系统来实现

## 用insmod方式测试



## 用proc方式测试

