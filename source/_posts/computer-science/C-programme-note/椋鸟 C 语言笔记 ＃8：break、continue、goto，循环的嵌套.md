---
date: 2023-10-20 23:01:28
title: 椋鸟C语言笔记#8：break、continue、goto，循环的嵌套
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

#### while 循环中的 break 与 continue

##### break

break 用于跳出当前循环

```c
#include <stdio.h>
 
int main()
{
    int i = 1;
 
    while(i<=10)
    {
        if(i == 5)
            break;//当i等于5后,就执行break,循环就终止了
        printf("%d ", i);
        i = i+1;
    }
    
    return 0;
}
```

上述程序只会打印 1 2 3 4,当 i 累积到 5 时就跳出了循环

##### continue

使用 continue 将跳过本次循环 continue 之后的代码

直接进行下一轮循环的判断

```c
#include <stdio.h>
 
int main()
{
    int i = 1;
 
    while(i<=10)
    {
        if(i == 5)
            continue;
//当i等于5后,就执行continue,直接跳过continue的代码,去循环的判断的地方 
//因为这里跳过了i++,所以i一直为5,程序陷入和死循环
    printf("%d ", i);
    i++;
    }
    
    return 0;
}
```

可以发现:如果循环变量的调整在**循环语句内且在 continue 后**可能就会死循环了

#### for 循环中的 break 与 continue

##### break

与 while 循环中的基本一致:

```c
#include <stdio.h>
 
int main()
{
    int i = 1;
    for(i=1; i<=10; i++)
    {
        if(i == 5)
            break;
        printf("%d ", i);
    }
    return 0;
}
```

##### continue

由于 for 循环中循环变量的调整在循环语句前

所以 continue 并**不会跳过循环变量的调整**,不易造成死循环:

```c
#include <stdio.h>
 
int main()
{
    int i = 1;
    for(i=1; i<=10; i++)
    {
        if(i == 5)
            continue;
//这里continue跳过了后边的打印,来到了i++的调整部分
        printf("%d ", i);
    }
    return 0;
}
```

#### 多种循环可以嵌套使用以达成复杂目的

输出 100-200 间的素数:

```c
#include <stdio.h>
 
int main()
{
    int i = 0;
    
    //循环产生100~200的数字 
    for(i=100; i<=200; i++)
    {
        //判断i是否为素数
        //循环产生2~i-1之间的数字 
        int j = 0;
        int flag = 1;//假设i是素数 
 
        for(j=2; j<i; j++)
        {
            if(i % j == 0)
            {
                flag = 0;
                break;
            }
        }
 
        if(flag == 1)
            printf("%d ", i);
    }
 
    return 0;
}
```

#### goto

goto 语句可以设置在**同一个函数内**(主函数也是函数哦)跳转到设置好的标号处:

```c
#include <stdio.h>
 
int main()
{
    printf("1\n");
    goto next;
    printf("2\n");
next:
    printf("3\n");
    return 0;
}
```

上方程序只打印 1 和 3

> **我们一般不使用 goto 语句,因为滥用 goto 语句将会导致逻辑混乱,导致系统崩溃等问题**

但有时使用 goto 也不错:

```c
for(...)
{
    for(...)
    {
        for(...)
        {
            if(disaster)
                goto error;
        }
    }
}
 
error:
    处理;
```

借助 goto 可以快速的跳出多重循环