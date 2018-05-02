---
title: linux下不重编译替换同名函数--LD_PRELOAD
category: 编程
date: 2018-05-02 20:07:57
tags: [linux,hack]
  
---

## LD_PRELOAD：利用动态链接库更改同名函数的功能

linux 的程序在运行时调用的函数是以符号的形式存储在静态链接库.a文件或动态链接库.so文件中的，Linux优先查找动态链接库中的符号，如果没有再去查找静态链接库。LD_PRELOAD是linux下的一个环境变量，设置了之后，linux会优先查找LD_PRELOAD路径中的动态链接库，然后再查找默认的动态链接库，再查找静态链接库。知道了这个特性，我们就可以在不修改程序代码的情况下，通过LD_PRELOAD设置，实现程序优先调用我们自己编译的动态链接库，从而更改原有的程序代码功能。

## LD_PRELOAD 实现样例

举个栗子：一个登录函数，输入密码正确后正常登录，错误返回登录失败。输入的密码与存储的密码比较函数假设原型为_int mycompare(const char * input, const char * origin)_, 当input 等于origin时返回登录成功，返回值为0，否则返回失败。如果知道了这个函数的原型，我们即可以利用LD_PRELOAD功能，自己构造一个mycompare函数，构造的这个函数比较任何数据都返回成功，这样我们就实现了破解的功能。

```c
/* ld_preload.c 主函数 */
#include <stdio.h>
extern int mycompare(const char *a, const char *b);
int main(int argc, char *argv[])
{
    char * passwd = "pass";
    if (argc != 2)
    {
        printf("wrong arg num, must be 2! \n");
        return 0;
    }

    if (0 == mycompare(passwd, argv[1]))
    {
        printf("correct, it is 'pass'\n");
    }
    else
    {
        printf("wrong, you input : %s, the right is %s\n",argv[1], passwd);
    }
    return 0;
}

```

比较函数代码如下，此函数编译成动态链接库供主函数调用。

```c
/* compare.c 生成.so文件 */
#include <stdio.h>
#include <string.h>
int mycompare(const char *a, const char *b)
{
    return strcmp(a,b);
}
```

编译步骤如下：

> gcc compare.c -fPIC -shared -o libcompare.so
>
> gcc ld_preload.c -L . -lcompare -o ld_preload

测试结果如下：

> $ ./ld_preload pass
>
> correct, it is 'pass'
>
> $ ./ld_preload a
>
> wrong, you input : a, the right is pass

现在我们新建一个myhack.c，重新实现mycompare函数

```c
/* myhack.c */
#include <stdio.h>
#include <string.h>
int mycompare(const char * a, const char * b)
{
    return 0; // hack, any passwd is right.
}
```

编译处理步骤如下，处理过后任意输入字符都会匹配输出相等，达到了替换原功能函数的目的，用unset LD_PRELOAD可以恢复原先设置：

> $ gcc myhack.c -fPIC -shared -o libmyhack.so
>
> $ export LD_PRELOAD="./libmyhack.so"
>
> $ ./ld_preload a 
>
> correct, it is 'pass'
>
> $ unset LD_PRELOAD
>
> $ ./ld_preload a
>
> wrong, you input : a, the right is pass

## 参考知识点

| gcc 编译选项    | 含义                                                         |
| --------------- | ------------------------------------------------------------ |
| -c              | 只编译不链接，生成目标文件“.o"                               |
| -S              | 只编译不汇编，生成汇编代码                                   |
| -E              | 只进行预编译，不做其他处理                                   |
| -g              | 在可执行程序中包含标准调试信息                               |
| -o file         | 指定输出文件为file                                           |
| -v              | 打印出编译器内部编译过程命令行信息和编译喊叫版本             |
| -I dir          | 在头文件的搜索路径列表中添加 dir 目录                        |
| -static         | 静态编译，生成.a文件                                         |
| -shared         | 1、生成动态库文件 <br />2、优先链接动态库，无动态库才链接表态库 |
| -L dir          | 在库文件搜索路径中加入dir                                    |
| -lname          | 链接名称为libname.a或libname.so的库文件，<br />如果都有时根据选项-static或-shared来选择 |
| -fPic (或-fpic) | 生成使用相对地址（position indepedent code），<br />生成位置无关的目标代码，通常生成动态链接库时使用 |

## 一些编译问题

**1、如何查看一个程序中链接用到的库**

> 用ldd 命令
>
> $ ldd ld_preload
> linux-gate.so.1 =>  (0xb7f01000)
> libcompare.so => not found
> libc.so.6 => /lib/i386-linux-gnu/libc.so.6 (0xb7d31000)
> /lib/ld-linux.so.2 (0xb7f03000)

**2、gcc -L 指定路径后还是找不到.so文件**

> ubuntu有的是有这个问题，当时我这边是用这个命令设置了一下链接库路径就好了，故障时如上一问题所示
> $ export LD_LIBRARY_PATH=.
> $ ldd ld_preload
> linux-gate.so.1 =>  (0xb7f0b000)
> libcompare.so => ./libcompare.so (0xb7f04000)
> libc.so.6 => /lib/i386-linux-gnu/libc.so.6 (0xb7d38000)
> /lib/ld-linux.so.2 (0xb7f0d000)


