---
date: 2023-11-24 12:15:56
title: 椋鸟C语言笔记#22：回调函数、泛式编程、qsort
category: 
  - [计算机科学, C语言笔记]
tags:
  - C
  - 函数
  - 算法
---

:::info
该系列为本人的学习笔记，主要由本人整理书写而成。部分内容来自教材、视频课程等，不能保证完全原创性。
:::

萌新的学习笔记，写错了恳请斧正。

#### 回调函数

当一个函数 a 的参数含有函数指针时，这个函数指针在使用时所指向的函数 b（被该函数调用的函数）就是回调函数。注意回调函数并不是由函数 a 决定调用的函数，而是在调用函数 a 时决定使用的函数。

比方说，上一篇笔记中的计算器如果不用转移表实现，也可以这样：

```c
#include <stdio.h>
 
int add(int a, int b)
{
    return a + b;
}
int sub(int a, int b)
{
    return a - b;
}
int mul(int a, int b)
{
    return a * b;
}
int div(int a, int b)
{
    return a / b;
}
void calc(int(*pf)(int, int))
{
    int ret = 0;
    int x, y;
    printf("输⼊操作数：");
    scanf("%d %d", &x, &y);
    ret = pf(x, y);
    printf("ret = %d\n", ret);
}
 
int main()
{
    int input = 1;
    do
    {
        printf("************************");
        printf("**1:add**2:sub**********");      
        printf("**3:mul**4:div**0:exit**");         
        printf("************************");
        printf("请选择：");
        scanf("%d", &input);
        switch (input)
        {
        case 1:
            calc(add);
            break;
        case 2:
            calc(sub);
            break;
        case 3:
            calc(mul);
            break;
        case 4:
            calc(div);
            break;
        case 0:
            printf("退出程序\n");
            break;
        default:
            printf("选择错误\n");
            break;
        }
    } while (input);
    return 0;
}
```

上方的 calc 函数的参数就是函数指针，在**实际调用 calc 函数时提供的指针指向的函数就是回调函数**

#### 泛式编程

泛式编程就是在编程的时候**尽量减少对特定数据类型的依赖**。

比方说我们要写一个冒泡排序的函数，如下：

```c
void sortofup(int s[], int n)
{
	_Bool flag = true;
	while (flag)
	{
		flag = false;
		for (int i = 0; i < n - 1; i++)
			if (s[i] > s[i + 1])
			{
				s[i] ^= s[i + 1];
				s[i + 1] ^= s[i];
				s[i] ^= s[i + 1];
				flag = true;
			}
	}
}
```

上面就是对一个长度为 n 的数组的冒泡排序的实现

但是这个数组只能是整型数组。如果我们需要对浮点型数组排序，就需要再写一个浮点型数组的冒泡排序；如果我们要对长整型数组排序，就需要再写一个长整型数组的冒泡排序。这就**十分死板**。

而在库函数中，有一个函数叫 **qsort**，它能**对任意类型的数据进行快速排序，十分灵活**

这就是泛式编程的应用，那么具体是怎么实现的呢？不急，待会讲。

#### qsort

##### qsort 是什么

qsort 是 stdlib.h（或 search.h）头文件中定义的一种排序函数

qsort 没有返回值（void），而其参数有 4 个：

1.  **void* base**：指向需要排序的数组的首元素的指针 base（注意前面笔记讲过 void * 类型的参数可以接受任意类型的指针传参，但是使用时需要先强制类型转换）
2.  **size_t num**：需要排序的数据的数量
3.  **size_t width**：需要排序的数据中每个数据占多少字节
4.  **int (*compare)(void*, void*)**：一个函数指针 compare，**指向一个返回值为整型参数为两个泛型指针变量的函数**。这就是比较函数。

##### 比较函数怎么写

在使用 qsort 时，我们需要排序某种类型的数据，就**先写一个对该类型数据的比较函数**。这个函数的参数是两个泛型指针，返回值是整型，而**返回值的正负或零代表了比较的结果是大于小于或等于（第一个参数更大时返回正数则递增排序，反之则为递减排序，如果有其他比较排序规则也可以自行添加，比如奇偶交叉等）**。比方说，比较两个整型数据，我们可以这么写：

```c
int Comp_int(const void* a, const void* b)
{
    int ia = *(int*)a;
    int ib = *(int*)b;
	if (ia > ib)
        return 1;
    if (ia < ib)
        return -1;
    else
        return 0;
}
```

上方代码我们将 a 与 b 强制类型转换为整型指针并解引用，这样我们就得到了整型 a 与 b 的值，并可以加以比较。

注意**如果指针传参时我们不希望原值被改变，无论函数中有没有会对原值产生影响的内容，我们都最好在参数前加一个 const 进行限制**，这是一个好习惯。 

我们也可以进一步根据实际情况简化比较函数：

```c
int Comp_int(void* a, void* b)
{
	return *(int*)a - *(int*)b;
}
```

##### 使用 qsort 排序各种数据

```c
#include <stdio.h>
#include <stdbool.h>
#include <stdlib.h>
#include <string.h>
 
struct Stu
{
	char Name[20];
	short ID;
};
 
int Comp_int(void* a, void* b)
{
	return *(int*)a - *(int*)b;
}
 
int Comp_flt(void* a, void* b)
{
	int c = 0;
	if ((c = *(float*)a - *(float*)b) == 0)
		return 0;
	return c > 0 ? 1 : -1;
}
 
int Comp_struct_by_ID(void* a, void* b)
{
	return ((struct Stu*)a)->ID - ((struct Stu*)b)->ID;
}
 
int main()
{
    //@@@@@排序整型数组并打印@@@@@
	int arr_int[] = { 2,5,78,3,31,6,7865,34,1,7,8,132,4,7,5 };
	size_t len_int = sizeof arr_int / sizeof arr_int[0];
	qsort(arr_int, len_int, sizeof arr_int[0], Comp_int);
	for (int i = 0; i < len_int; i++)
		printf("%d ", arr_int[i]);
	printf("\n");
 
    //@@@@@排序浮点型数组并打印@@@@@
	float arr_flt[] = { 12.5,3.5,6.2,9,1.1,98.2,0.4,-9.3,14.7 };
	size_t len_flt = sizeof arr_flt / sizeof arr_flt[0];
	qsort(arr_flt, len_flt, sizeof arr_flt[0], Comp_flt);
	for (int i = 0; i < len_flt; i++)
		printf("%.2f ", arr_flt[i]);
	printf("\n");
 
    //@@@@@排序结构体数组并打印@@@@@
	struct Stu arr_struct[] = { {"zhangsan",2},{"lisi",1},{"wangwu",3} };
	size_t len_struct = sizeof arr_struct / sizeof arr_struct[0];
        //按id排序
	qsort(arr_struct, len_struct, sizeof arr_struct[0], Comp_struct_by_ID);
	for (int i = 0; i < len_struct; i++)
		printf("%s %hd ", arr_struct[i].Name, arr_struct[i].ID);
	printf("\n");
        //按首字母排序
	qsort(arr_struct, len_struct, sizeof arr_struct[0], strcmp);
	for (int i = 0; i < len_struct; i++)
		printf("%s %hd ", arr_struct[i].Name, arr_struct[i].ID);
	printf("\n");
 
 
	return 0;
}
```

上方代码按首字母排序的 strcmp 函数后面笔记会讲，是 string.h 提供的函数

##### 模仿 qsort 写一个万能冒泡排序

我们可以模仿 qsort 来实现一个万能的冒泡排序

只需要做三件事：

*   将其中**比较数据大小的地方替换为比较函数的函数指针**
*   将指针变成 **char * 类型**（只占一个字节），然后利用数据数量与数据长度来**定位**指定数据
*   将交换数据的地方处理为**逐字节替换**（利用第二点的定位）

具体实现如下：

```c
#include <stdio.h>
#include <stdbool.h>
#include <string.h>
 
struct Stu
{
	char Name[20];
	short ID;
};
 
int Comp_int(void* a, void* b)
{
	return *(int*)a - *(int*)b;
}
 
int Comp_flt(void* a, void* b)
{
	int c = 0;
	if ((c = *(float*)a - *(float*)b) == 0)
		return 0;
	return c > 0 ? 1 : -1;
}
 
int Comp_struct_by_ID(void* a, void* b)
{
	return ((struct Stu*)a)->ID - ((struct Stu*)b)->ID;
}
 
 
void bsort(void* base, size_t count, size_t size, int(*comp_f)(void*, void*))
{
	int flag = 1;
	for (int k = 0; k < count; k++)
	{
		flag = 1;
		for (int i = 0; i < count - k - 1; i++)
		{
			if (comp_f((char*)base + size * i, (char*)base + size * (i + 1)) > 0)
			{
				for (int j = 0; j < size; j++)
				{
					char* a = (char*)base + size * i + j;
					char* b = (char*)base + size * (i + 1) + j;
					char c;
					c = *a, *a = *b, *b = c;
				}
				flag = 0;
			}
		}
		if (flag)
			break;
	}
}
 
 
int main()
{
	int arr_int[] = { 2,5,78,3,31,6,7865,34,1,7,8,132,4,7,5 };
	size_t len_int = sizeof arr_int / sizeof arr_int[0];
	bsort(arr_int, len_int, sizeof arr_int[0], Comp_int);
	for (int i = 0; i < len_int; i++)
		printf("%d ", arr_int[i]);
	printf("\n");
 
	float arr_flt[] = { 12.5,3.5,6.2,9,1.1,98.2,0.4,-9.3,14.7 };
	size_t len_flt = sizeof arr_flt / sizeof arr_flt[0];
	bsort(arr_flt, len_flt, sizeof arr_flt[0], Comp_flt);
	for (int i = 0; i < len_flt; i++)
		printf("%.2f ", arr_flt[i]);
	printf("\n");
 
	struct Stu arr_struct[] = { {"zhangsan",2},{"lisi",1},{"wangwu",3} };
	size_t len_struct = sizeof arr_struct / sizeof arr_struct[0];
	bsort(arr_struct, len_struct, sizeof arr_struct[0], Comp_struct_by_ID);
	for (int i = 0; i < len_struct; i++)
		printf("%s %hd ", arr_struct[i].Name, arr_struct[i].ID);
	printf("\n");
	bsort(arr_struct, len_struct, sizeof arr_struct[0], strcmp);
	for (int i = 0; i < len_struct; i++)
		printf("%s %hd ", arr_struct[i].Name, arr_struct[i].ID);
	printf("\n");
 
	return 0;
}
```