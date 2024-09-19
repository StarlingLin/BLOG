---
date: 2023-10-20 23:01:28
title: 椋鸟C语言笔记#6：用switch函数构建分支语句
category: 
  - [计算机科学, C语言笔记]
tags:
  - C
  - 语句
---

:::info
该系列为本人的学习笔记，主要由本人整理书写而成。部分内容来自教材、视频课程等，不能保证完全原创性。
:::

萌新的学习笔记,写错了恳请斧正。

#### Switch 语句的用法

switch 语句其实就是一种特殊的 if...else... 语句

但在有些情况下,用 switch 语句更直观

```c
switch (expression) 
{
case value1: statement1; //case和valve间应有空格
case value2: statement2;
default: statement3;
}
```

switch 语句会根据 expression 表达式的值选择一个分支,并顺次执行

如果找不到对应的值,则从 default 分支开始顺次执行

> 要点 1:**expression 必须是整型表达式,value 必须是整型常量**
> 
> 要点 2:选择分支后**顺次执行**,而不是只执行这一项。
> 
> 比方说 expression 的值与 value2 匹配,则先执行 statement2,再 statement3

#### switch 语句中的 break

如果我们不想让 switch 选完分支后顺次执行

而是只执行这一项,该怎么办呢?

```c
#include <stdio.h>
 
int main()
{
    int n = 0;
 
    scanf("%d", &n);
 
    switch(n%3)
    {
    case 0: printf("整除,余数为0\n"); break;
    case 1: printf("余数是1\n"); break;
    case 2: printf("余数是2\n"); break;
    }
 
    return 0;
}
```

如上例,只要在语句后加一个 break,便可直接跳出该结构

而如果没有 break,就可能出现奇怪的情况:

![奇怪的情况](1.png "奇怪的情况")

#### 有效的利用 break

我们不一定要每一个分支后都加上 break

有时不加 break 也是一种方法:

```c
#include <stdio.h>
 
int main()
{
    int day = 0;
 
    scanf("%d", &day);
 
    switch(day)
    {
    case 1:
    case 2:
    case 3:
    case 4:
    case 5: printf("工作日\n"); break;
    case 6:
    case 7: printf("休息日\n"); break;
    }
    
    return 0;
 
}
```

#### switch 语句里的 default

default 语句一定要放在最后吗?

**不一定**

case 和 default 没有顺序要求,但是一般把 default 放最后