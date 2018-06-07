---
title: linux 写时拷贝技术
tags: [linux]
category: 宋宝华
date: 2018-05-29 22:31:08
---

## 同一个全局变量显示不同的值

如下代码所示，子进程中打印全局变量data后，再对data进行修改，父进程在等待1s（确保子进程已修改完成）后再打印data的值。

```c
#include <stdio.h>
#include <sys/types.h>
#include <unistd.h>
int data = 10;
int child_process()
{
    printf("child process %d, data %d\n", getpid(), data);
    data = 20;
    printf("child process %d, data %d\n", getpid(), data);
    _exit(0);
}
int main(int argc, char * argv[])
{
    int pid;
    pid = fork();
    if(pid == 0){
        child_process();
    }
    else{
        sleep(1);
        printf("parent process %d, data %d", getpid(),data);
        _exit(0);
    }
    return 0;
}
```

```bash
#运行程序的显示结果如下
child process 9491, data 10
child process 9491, data 20
parent process 9490, data 10
```

在正常情况下，程序修改全局变量data，再打印data，会是修改后的值 20，但由于进程采用了COW（copy-on-write)技术，使得同一个变量在不同进程显示不同的值。

COW技术必须借助MMU（内存管理单元）来实现。具体流程如下图所示