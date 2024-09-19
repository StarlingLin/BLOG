---
date: 2023-12-11 14:16:26
title: 椋鸟C语言笔记#27：字符串数字提取（atoi、atol、atoll、atof）
category: 
  - [计算机科学, C语言笔记]
tags:
  - C
  - 函数
---

:::info
该系列为本人的学习笔记，主要由本人整理书写而成。部分内容来自教材、视频课程等，不能保证完全原创性。
:::

萌新的学习笔记，写错了恳请斧正。

在 stdlib.h 中还有几个有意思的字符串函数

它们的功能是将字符串开头的数字提取出来

下面我们具体看一看这几个函数吧

#### atoi

```c
#include <stdlib.h>
int atoi(const char* string);
```

atoi，即 auto int。舍弃任何空白符，直至找到首个非空白符，然后接收尽可能多的字符以组成合法的整数（可以有负号）表示，并转换之为整数值。如果没有读取到数字，就返回 0；如果超出可读取类型范围，那么返回值是未定义的。

使用示例如下：

```c
#include <stdio.h>
#include <stdlib.h>
 
int main(void)
{
    printf("%i\n", atoi(" -123junk"));
    printf("%i\n", atoi("0"));
    printf("%i\n", atoi("junk"));         // 无可进行的转换
    printf("%i\n", atoi("2147483648"));   // 在 int 范围外
}
```

输出：

```c
-123
-2147483648
```

##### 模拟实现

```c
#include <stdio.h>
#include <ctype.h>
#include <assert.h>
 
int my_atoi(const char* str)
{
	assert(str);
	int ret = 0, sign = 1, flag = 1;
	while (isspace(*str++));
	str -= 2;
	while (isdigit(*++str) || '-' == *str && flag || '+' == *str && flag)
		if (*str == '-')
			flag = 0, sign = -1;
		else if (*str == '+')
			flag = 0, sign = 1;
		else
			flag = 0, ret = 10 * (ret + *str - '0');
	
	return sign * ret / 10;
}
 
int main()
{
	char str[] = "	 -114514";
	int ret = my_atoi(str);
	printf("%d", ret);
	return 0;
}
```

#### atol 与 atoll（C99 起）

与 atoi 功能完全一致，只是变成 long 和 long long 类型

```c
#include <stdlib.h>
long atol(const char* string);
long long atoll(const char* string);
```

#### atof

atof 用于提取的是浮点数（小数形式或者科学计数法）

atof 函数使用需要包含 stdlib.h 和 math.h 两个头文件

```c
#include <stdlib.h>
#include <math.h>
double atof(const char* string);
```

函数会舍弃任何空白符，直至找到首个非空白符。然后它会取用尽可能多的字符，以构成合法的浮点数表示，并将它们转换成浮点值。

##### 合法的浮点值

1. 十进制浮点数表达式。它由下列部分组成：

*   (可选) 正或负号
*   非空的十进制数字序列，可选地包含一个小数点字符（由当前的 C 本地环境确定）（定义有效数字）
*   (可选) `e` 或 `E` ，并跟随可选的正或负号，以及非空十进制数字序列（以 10 为底定义指数）

2. 十六进制浮点数表达式 (C99起)。它由下列部分组成：

*   (可选) 正或负号
*   `0x` 或 `0X`
*   非空的十六进制数字序列，选地包含一个小数点字符（由当前的 C 本地环境确定）（定义有效数字）
*   (可选) `p` 或 `P` ，并跟随可选的正或负号，以及非空十进制数字序列（以 2 为底定义指数）

3. 无穷大表达式 (C99 起)。它由下列部分组成：

*   (可选) 正或负号
*   `INF` 或 `INFINITY` ，忽略大小写

4. 非数（NaN）表达式 (C99 起)。它由下列部分组成：

*   (可选) 正或负号
*   `NAN` 或 `NAN(`_char_sequence_`)` ，忽略 `NAN` 部分的大小写。 _char_sequence_ 只能由数字、拉丁字母和下划线构成。结果是一个静态的 NaN 浮点值。

5. 任何其他可由当前 C 本地环境接受的表达式

##### 返回值

成功时返回代表 string 内容的 double 值

若转换的值在返回值范围外，则返回值未定义

若无可进行的转换，则返回 0.0

##### 使用示例

```c
#include <stdlib.h>
#include <stdio.h>
 
int main(void)
{
    printf("%g\n", atof("  -0.0000000123junk"));
    printf("%g\n", atof("0.012"));
    printf("%g\n", atof("15e16"));
    printf("%g\n", atof("-0x1afp-2"));
    printf("%g\n", atof("inF"));
    printf("%g\n", atof("Nan"));
    printf("%g\n", atof("1.0e+309"));   // 超出 double 范围
    printf("%g\n", atof("0.0"));
    printf("%g\n", atof("junk"));       // 无可进行的转换
}
```

在某种环境下输出：

```c
-1.23e-08
0.012
1.5e+17
-107.75
inf
nan
inf
```