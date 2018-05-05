---
title: 字符处理函数的使用
category: 编程
date: 2018-05-03 21:11:13
tags: [linux,c,api]
---

## 字符转换函数的使用场景

我们在程序界面输入数据时，基本上都是用字符来保存的，所有操作系统之间的信息保存和传递也是基于字符（char）单位来进行，单位再大了就有字节序问题，不能兼容所有的操作系统了。为效率考虑，现在的信息全以字符为单位。计算一段数据的大小，比如一个变量Int的大小，用sizeof操作符返回的结果就是字节数，也就是char的个数。sizeof(int)  == 4 ，在程序应用中，比如在计数器中我们输入一串数字和符号  123 + 456 ，内部如何计算出结果呢，123在输入时是对应3个字符char，在计算的时时候我们则必须将其转换成整型变量数据再进行加法运算。C函数提供的一个函数库API是atoi, ascii to integer 的一个函数 stdlib.h，其原型是int atoi(const char * ptr)，函数开始扫描ptr字符串，跳过空白字符(isspace检测)，从数字或正负号开始，到\0或非数字结束，返回一个整数，当无法转换成整理或ptr是空字符串时返回0

同样的，也会有itoa函数，将整数转换成字符串，但奇怪的是这个函数却不是标准的C函数，在Linux下基本没有这个函数，通常windows下的非标准编译器才支持这种函数，我考虑了多方面因素，可能是以下原因，转换出的字符串一般都是一串不确定长度的空间，而这个内存空间只能通过指针的方式返回使用者，这样使用者调用后还需要手动释放这个空间，很容易造成内存泄露。linux下将数字转换成字符有以下几种方式

> ## int sprintf( char \* buffer,  char \* fmt , ...);
>

sprintf最常见的应用之一莫过于把整数打印到字符串中，所以，spritnf 在大多数场合可以替代itoa。如：

1. 将整数转成字符串

   ```c
   sprintf (s, "%d", 123); // 产生"123"
   ```

2. 连接字符串

   ```c
   char *who = "I";
   char *whom = "52PHP"; 
   sprintf(s, "%s love %s.", who, whom); // 产生："I love 52PHP. "
   ```

   *spintf 常见问题*

   1、返回值返回的是实际写入buffer的字符长度，可以不用再用strlen计算字符长度了

   2、如果 s 的区域不足，写入的数据超过s长度，可能会导致内存访问错误引起崩溃，

> ## int snprintf(char \*buf, size_t n, char \* fmt, ...)
>

为了防止引起内存访问错误，采用snprintf函数，此函数最多从源字符串中取n-1个字符，最后一个字节补\0，写入buf，推荐用法是将第2个参数n写成sizeof(buf)，与sprintf不同的时返回值 int 是源字符串的字符数，而非实际写入buf的字符串数目。在Linux内核中不推荐用snprintf，就是因为返回的值可能大于实际写入的buf中的字符数目，可能导致计算时越界，linux内存中推荐采用返回值为实际写入buf中的字符数字函数scnprintf，用法和snprintf一样，但返回值是实际写入buf的字符数，这样就避免了返回值比实际写入buf数值还大的现象。

> ## int scnprintf(char \*buf, size_t n, char \*fmt, ...)
>

参考文章：https://www.kernel.org/doc/htmldocs/kernel-api/API-scnprintf.html