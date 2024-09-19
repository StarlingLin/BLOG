---
date: 2023-11-23 20:35:42
title: 椋鸟C语言笔记#21：函数指针、函数指针数组(转移表)、typedef及其必要性
category: 
  - [计算机科学, C语言笔记]
tags:
  - C
  - 数组
  - 指针
---

:::info
该系列为本人的学习笔记，主要由本人整理书写而成。部分内容来自教材、视频课程等，不能保证完全原创性。
:::

萌新的学习笔记,写错了恳请斧正。


#### 函数指针

函数指针,顾名思义,就是存放函数地址的指针

##### 函数的地址

函数也有地址吗?我们来看一段代码:

```c
#include <stdio.h>
 
void func()
{
    ;
}
 
int main()
{
    printf(" func = %p\n", func );
    printf("&func = %p\n", &func);
    return 0;
}
```

在一次运行中其结果为:

```c
func = 00007FF60C0211A4
&func = 00007FF60C0211A4
```

可见,函数是有地址的。而且**和数组一样,可以用取地址操作符获取其地址,而且函数名本身也代表了其地址**。那我们得到了函数的地址有什么用呢?

我们可以通过地址来调用函数,这部分下面讲。

##### 函数指针的创建

```c
int (*pf) (int, float);
int (*pf) (int a, float b);    //一个意思,都行
```

上方就是创建了一个名叫 pf 的函数指针,而这个函数的返回值就是前面的 int,函数的变量为后面括号里的 int 与 float。其实我们可以发现,**定义函数指针的语句与声明函数的语句的区别只有在函数名 / 函数指针名外面包上一个括号并加一个星号**

而这个函数指针变量的变量类型是:

`int (*) (int, float)`

##### 函数地址的使用

既然得到了函数的地址,我们就可以用地址来调用函数。其实这里写法与直接调用函数类似,或者不如说直接调用函数就可以理解为函数名是一个函数指针:

```c
#include <stdio.h>
 
int test()
{
    return 1;
}
 
int main()
{
    int a = test();
    int (*pf)() = &test;
    int b = pf();
    printf("%d", a);
    printf("%d", b);
    return 0;
}
```

这里输出结果 a=b=1,我们发现函数指针可以直接用来调用函数。

这具体能给我们的代码带来什么变化呢?下面讲。

#### typedef 及其必要性

##### typedef

typedef 可以用来给类型名起 “绰号”,让代码更易读易写:

`typedef unsigned long long ull;`

这里就是给 unsigned long long 起了一个叫 ull 的 “绰号”,之后我们就能直接使用 ull 了,比如直接定义 unsigned long long 类型的变量:

`ull a = 1145141919810;`

我们也可以给指针类型和数组类型起 “绰号”:

```c
typedef int* pt_i;
typedef int[5] pt_arr;
```

同样的,也可以给指针数组、数组指针 “起绰号”,但是复杂一点。

我们假设指针数组的类型是 (int(*)[9]),数组指针的类型是 (int*[9])

我们**需要像定义这两种类型变量一样,把 “别名” 放在原本变量名的位置**,如:

```c
typedef int (*type1)[9];
typedef int *type2[9];
```

这里就成功的起了 type1 和 type2 两个 “绰号”。

那么再试试给函数指针 “起绰号”,其过程和数组指针类似

我们假设函数指针的类型是 (int (*) (int, float)):

`typedef int (*type3) (int, float);`

##### typedef 的必要性

我们先看个语句:

`(*(void (*)())0)();`

这个代码十分晦涩,易读性很差。

那如果写成这样呢:

```c
typedef void(*type_pfun)();
type_pfun pfun;
pfun = (type_pfun)0;
(*pfun)();
```

我们可以看到,这四条语句就好理解多了:

> 第一行将一个参数为空返回为空函数指针的类型起绰号为 type_pfun
> 
> 第二行定义了一个 type_pfun 类型的函数指针 pfun
> 
> 第三行将 0 强制类型转换为 type_pfun, 并赋值给 pfun
> 
> 第四行解引用 pfun,调用这个无输入无输出的函数
> 
> 四行合起来就是去调用一个 0 地址处的无参数无输出的函数

这里就能明显的看出 typedef 对代码易读性的重要作用了。

再看一个:

`void (*signal(int , void(*)(int)))(int);`

同样的,可以把代码拆为:

```c
typedef void (*type_pfun)(int);
type_pfun signal(int, type_pfun);
```

这样,我们明显能看出,其实就是声明了一个叫 signal 的函数。这个函数的返回类型是一个函数指针,参数也有一个函数指针。而这两个函数指针都指向了有一个 int 类型参数无返回值的函数。

#### 函数指针数组(转移表)

函数指针数组,也叫转移表,就是存放函数指针的数组。

和前面指针数组的逻辑类似,要**让最终定义出来的是数组,就需要让变量名先与方括号结合**:

`int (*pfunarr[6])();`

这就定义了一个叫 pfunarr 的数组,数组长度 6,存放 (int(*)()) 类型的函数指针

其使用方式与正常的数组一致,我们可以利用其实现在不同条件下调用不同的函数

比方说,在用户选择的模式不同时进行不同计算的计算器: 

```c
#include <stdio.h>
 
typedef int (*f_calc)(int, int);
 
void menu()
{
	printf("*******************\n");
	printf("***1.ADD***4.DIV***\n");
	printf("***2.SUB***5.MOD***\n");
	printf("***3.MUL***0.exit**\n");
	printf("*******************\n");
}
 
int ADD(int a, int b)
{
	return a + b;
}
 
int SUB(int a, int b)
{
	return a - b;
}
 
int MUL(int a, int b)
{
	return a * b;
}
 
int DIV(int a, int b)
{
	return a / b;
}
 
int MOD(int a, int b)
{
	return a % b;
}
 
 
int main()
{
	int m = 0, n = 0, mode = 0;
	f_calc calc[6] = {NULL, ADD, SUB, MUL, DIV, MOD};
	menu();
	printf("Type in calc mode:");
	scanf("%d", &mode);
	printf("Type in the two calc number:");
	scanf("%d%d", &m, &n);
	printf("The answer is: %d\n", calc[mode](m, n));
	return 0;
}
```