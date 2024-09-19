---
date: 2023-10-18 11:00:47
title: 椋鸟C语言笔记#4：分支与循环
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

#### if 语句的语法

```c
if ( 表达式 )   //后面不要加分号
    语句;
```

如果表达式为真,就执行语句;反之不执行。

> C 语言中,0 表示假、非 0 的数表示真。因此只要表达式结果不等于 0 即执行语句。

#### else

if...else... 语句的用法如下

```c
if ( 条件 )
    语句1;
else
    语句2;
```

这样,如果条件成立,则执行语句 1;反之执行语句 2

那我们就可以写一个判断是否成年的程序:

```c
#include <stdio.h>
 int main()
 {
     int age = 0;
     scanf("%d", &age);
     if(age>=18)
         printf("成年\n");
     else
         printf("未成年\n");
 return 0;
 }
```

#### 嵌套 if

默认 if 和 else 语句只会控制一条语句,如果想控制更多语句可以用一对 “{}” 将其括起来。

括起来的许多语句就构成了一个语句块,也即复合语句。

这样,我们就可以把一个 if...else... 语句嵌套进另一个 if...else... 语句,构成三项选择。

```c
#include <stdio.h>
 
int main()
{
    int m;
    scanf("%d", &m)
    if (m < 0)
        printf("负数");
    else
    {
        if (m > 0)
            printf("正数");
        else
            printf("零");
    }
    return 0;
}
```

上方是一个判断正负与零的程序

但我们发现,这样写太麻烦了,如果有很多项的选择,就会使代码很臃肿

C 语言给我们提供了一个解决方案:

上一个 else 和下一个 if 可以连用,构成多重判断,如:

```c
#include <stdio.h>
 int main()
 {
    int age = 0;
    scanf("%d", &age);
    if(age<18)
        printf("少年\n");
    else if(age<=44)
        printf("⻘年\n");
    else if(age<=59)
        printf("壮年\n");
    else
        printf("老年\n");
    return 0;
 }
```

#### 悬空 else

else 总是遵循 “就近原则”

也就是说它会和上方最近的一个可匹配的 if 匹配,与缩进无关

若需隔项匹配需要适当添加 `{}`