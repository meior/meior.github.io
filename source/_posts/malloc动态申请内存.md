---
title: malloc动态申请内存
date: 2015-09-29 20:45:57
categories: C
tags: [Pointer]
---
前几天研究顺序栈，在为栈初始化时碰到一点问题。初始化使用stdlib.h库中的malloc函数动态分配内存，这是再熟悉不过的用法，但却出现了意想不到的问题：编译毫无问题，运行后就会提示停止运行，大概是内存分配的问题。下面来详细分析一下：

**_下述实验均在vc++ 6.0环境下进行_**
{%codeblock lang:c%}
#include<stdio.h>
#include<stdlib.h>
//定义一个存放整型数的栈结构体
typedef struct
{
    int num[100];
    int top;
}Istack;
//初始化
void setNullSatck(Istack* ss)
{
    ss = (Istack*)malloc(sizeof(Istack));
    ss->top = -1;
}
int main()
{
    Istack s;
    setNullSatck(&s);
    //打印初始化后的top
    printf("%d\n",s.top);
    return 0;
}
{%endcodeblock%}
这段代码在逻辑上看起来没有任何问题，实际上它编译也没有任何问题，猜猜程序运行会打印什么结果？出人意料，程序打印了“23”这个结果，显然这意味着初始化失败了，而实际上setNullStack函数是正确运行了的，问题在哪里？
<!--more-->
有人会想是不是指针的问题，传过去的参数不对，好，我们换一种方式，修改部分代码：
{%codeblock lang:c%}
//初始化
void setNullSatck(Istack* ss)
{
    ss = (Istack*)malloc(sizeof(Istack));
    ss->top = -1;
}
int main()
{
    Istack* s;
    setNullSatck(s);
    //打印初始化后的top
    printf("%d\n",s->top);
    return 0;
}
{%endcodeblock%}
同样编译没有错误，运行后直接停止运行，看来堆栈s并没有初始化成功，那么问题究竟在哪里？

实际上这还是参数传递的问题，主函数中调用setNullStack(s)，实际上是将指向栈的指针s的值复制了一份给形参ss，ss是一个独立的指针类型的变量，在函数中使用malloc为其动态分配内存并将top置为-1，只是初始化了局部指针变量ss，实际上指针s并未初始化，所以s->top也就子虚乌有了，程序自然会停止运行。而将s定义为Istack类型，将s的地址传过去(&s)道理也是一样的。以下例子可以说明这个解释：
{%codeblock lang:c%}
//我们直接在main函数中初始化，避免传递参数
int main()
{
    Istack* s;
    s = (Istack*)malloc(sizeof(Istack));
    s->top = -1;
    //打印初始化后的top
    printf("%d\n",s->top);
    return 0;
}
{%endcodeblock%}
此时运行后，程序正常打印出了初始化后top的值：-1。当然在后期算法中对堆栈操作频繁，所以初始化还是单独拿出来定义为一个函数比较方便，我们可以这样写：
{%codeblock lang:c%}
void setNullSatck(Istack* s)
{
    //利用*取到动态分配的内存空间
    *s = *(Istack*)malloc(sizeof(Istack));
    s->top = -1;
}
int main()
{
    Istack s;
    setNullSatck(&s);
    printf("%d\n",s.top);
    return 0;
}
{%endcodeblock%}
其实，如果你将s定义为Istack，初始化时不需要为其动态分配内存，于是我果断删掉了，这个玩意折腾了我好久，基础不牢，实在惭愧。