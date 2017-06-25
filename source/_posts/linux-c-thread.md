title: Linux下面使用C语言写多线程程序
date: 2014-04-13 22:49:13
tags: [并发, Linux, C]
categories: 编程语言
---
####一.概述
在Linux系统中编写多线程程序，需要使用Linux系统本身的一个库函数来创建线程

<!-- more -->

```c
int pthread_create(pthread_t *thread, const pthread_attr_t *attr,
                   void *(*start_routine) (void *), void *arg);
```
返回值

* 若线程创建成功，则返回0。若线程创建失败，则返回出错编号

参数

* 第一个参数为指向线程标识符的指针。
* 第二个参数用来设置线程属性。
* 第三个参数是线程运行函数的起始地址。
* 最后一个参数是运行函数的参数。

函数pthread_join用来等待一个线程的结束,创建线程后，可以在创建线程的主线程中使用这个函数来等待创建的线程运行结束
```c
int pthread_join(pthread_t thread, void **retval);
```
返回值

* 0代表成功。 失败，返回的则是错误号。

参数

* thread: 线程标识符，即线程ID，标识唯一线程。
* retval: 用户定义的指针，用来存储被等待线程的返回值。

####二.代码
```c
#include <stdio.h>
#include <unistd.h>
#include <pthread.h>

/** 两个线程函数的说明 **/
void *One(void * no);
void *Two(void * no);

int main(int argc, char **argv)
{
    /** 线程变量的定义 **/
    pthread_t A, B;

    /** 注意函数名One 和Two这里当作函数参数传入 **/
    pthread_create(&A, NULL, One, NULL);
    pthread_create(&B, NULL, Two, NULL);
    
    /** 主线程等待创建的两个子线程A,B运行结束后才退出，不然主线程退出后，
     *  创建的子线程执行不完也就退出了，pthread_join()函数，以阻塞的方式
     *  等待thread指定的线程结束。
     **/
    pthread_join(A, NULL);
    pthread_join(B, NULL);

    return 0;
}

void *One(void *no)
{
    int i;
    for(i = 0; i < 5; ++i)
    {
        printf("NUAA\n");
        // 等待1秒钟
        sleep(1);
    }
    
    return NULL;
}

void *Two(void *no)
{
    int i;
    for(i = 0; i < 5; ++i)
    {
        printf("CS\n");
        sleep(1);
    }
    
    return NULL;
}
```
注意

* 1.编译命令：gcc xx.c -o xx -l pthread，编译的时候必须带上-l pthread，不然编译不过。
* 2.执行./xx然后看到在控制台交替输出NUAA和CS
