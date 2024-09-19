---
date: 2024-01-20 06:10:55
title: 椋鸟C语言笔记#35：scanf/fscanf/sscanf、printf/fprintf/sprintf/snprintf，以及它们的边界检查函数
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

#### scanf/fscanf/sscanf

```c
#include <stdio.h>
int scanf( const char* format, ... );
int fscanf( FILE* stream, const char* format, ... );
int sscanf( const char* buffer, const char* format, ... );
```

**这三个函数从各种资源读取数据，按照格式控制字符串（format）转译，并将结果存储到指定位置**

**这三个函数返回成功赋值的接受参数的数量（可以为零），出错则返回 EOF。**

他们的区别在于：

*   scanf 从 stdin 读取数据。
*   fscanf 从文件流 stream 读取数据。
*   sscanf 从**以'\0'结尾**的字符串 buffer 中读取数据  。

scanf 和 fscanf 已经具体的讲过了

sscanf 的特性与 fscanf 等同，**抵达字符串尾（'\0'）等价于 fscanf 抵达文件尾的条件**

##### sscanf 使用实例

```c
#include <stdio.h>
#include <stdlib.h>
 
int main()
{
	int a = 0;
	char* arr = (int*)malloc(100 * sizeof(int));
	char str[100] = { 0 };
	sscanf(" 114514 afjoiweaj jiojl", "%d%s%s", &a, arr, str);
	printf("%d - %s - %s", a, arr, str);
	return 0;
}
```

#### printf/fprintf/sprintf/snprintf

```c
#include <stdio.h>
int printf( const char* format, ... );
int fprintf( FILE* stream, const char* format, ... );
int sprintf( char* buffer, const char* format, ... );
int snprintf( char* buffer, int bufsz, const char* format, ... );
```

**前三个函数从给定位置加载数据，按一定要求转换为字符串等价物，并写结果到各种地方**

**第四个函数 snprintf 就是增加了长度限制的 sprintf，其他没有区别**

对比四个函数：

*   printf 直接写结果到 stdout。返回打印的字符数，出错返回一个负值。
*   fprintf 写结果到流 stream。返回写入的字符数，出错返回一个负值。
*   sprintf 写结果到字符串 buffer，**结尾补上'\0'**。返回写入的字符数（**不包括空终止符'\0'**），出错返回一个负值。
*   snprintf 写结果到字符串 buffer，但是**至多写 bufsz-1 个字符**，结尾再加'\0'。**返回忽略 bufsz 限制情况下本应写入的字符数**，出错返回一个负值。如果 bufsz 为 0，则不写入任何内容；如果 buffer 是空指针，正常计算返回值但是不会真的写入内容。

##### sprintf 使用实例

```c
#include <stdio.h>
 
int main()
{
	char a[10] = "1919810";
	sprintf(a, "%s", "114514");
	printf(a);
 
	return 0;
}
```

##### snprintf 使用实例

```c
#include <stdio.h>
 
int main()
{
	char str[100];
	snprintf(str, 100, "Hello %s", "World!");
	printf(str);
	return 0;
}
```

#### scanf 系列的边界检查函数（scanf_s/fscanf_s/sscanf_s）

> **首先声明一点，这些为了安全创造的带有下划线 s 结尾的边界检查函数是在 C11 标准中定义的****可选****函数，所以并不是每一个编译器都能编译这个函数，只有在使用的编译器实现定义 “__STDC_LIB_EXT1__” 而且在包含 < stdio.h > 前定义 “__STDC_WANT_LIB_EXT1__” 为整数常量 1 时才能保证这些函数一定可以使用。当然，不少编译器都默认开启了这些函数，比如在 Visual Studio 中这些函数就是默认开启的，甚至如果使用不安全的不带边界检查的这些函数还会警告并中断编译（在文件前定义 “_CRT_SECURE_NO_WARNINGS” 为整数常量 1 可以忽略不安全函数警告）**
> 
> ```c
> //就是加上这一行以禁用不安全函数警告
> #define _CRT_SECURE_NO_WARNINGS 1
> //后面的1可以省略，默认定义值为1
> ```

这些函数相比于传统函数，多了边界检查，可以防止缓冲区溢出

```c
#include <stdio.h>
int scanf_s(const char* format, ...);
int fscanf_s(FILE* stream, const char* format, ...);
int sscanf_s(const char* buffer, const char* format, ...);
```

机制与传统函数相同，除了：

**1.** 如果在格式限定中使用了 %c 、 %s 或 %[，后面**可变参数列表中与其对应的参数变成两个**，第一个仍为传统函数的参数，第二个则是**类型为 rsize_t** 的一个值，用于限定这个格式匹配符最多能匹配多少个字节的内容。比方说：

```c
#include <stdio.h>
 
int main()
{
	int n = 0, m = 0;
	char arr[10] = { 0 };
	scanf_s("%d%x%s", &n, &m, arr, (rsize_t)sizeof arr);
	printf("n=%d m=%d arr=%s\n", n, m, arr);
	return 0;
}
```

这段代码中，scanf_s 中 %s 对应的参数包括 arr 和 (rsize_t)sizeof arr 两个 arr 是读取后保存的位置，后面则是限定最多读取 10 个字节（**空终止符'\0'是算在内的**，所以实际最多读 9 个）

**2.** 在运行时额外检测以下违规行为，如果出现违规也返回 EOF：

*   任何指针类型的参数是空指针
*   format、stream、buffer 中存在空指针
*   使用 %c 、 %s 或 %[ 会写入的字符数，加上空终止字符'\0'，要超过我们通过参数给定的限制
*   可选的其他错误，因编译器而异

#### printf 系列的边界检查函数（printf_s/fprintf_s/sprintf_s/snprintf_s）

其使用条件与优越性与 scanf 系列的边界检查函数相同

```c
#include <stdio.h>
int printf_s(const char* format, ...);
int fprintf_s(FILE* stream, const char* format, ...);
int sprintf_s(char* buffer, rsize_t bufsz, const char* format, ...);
int snprintf_s(char* buffer, rsize_t bufsz, const char* format, ...);
```

其机制与传统函数相同，除了在运行时检测如下违规行为：

*   format 中存在转换说明符 %n
*   任何对应 %s 的参数是空指针
*   format 或 buffer 是空指针
*   bufsz 为 0 或大于其类型限制（RSIZE_MAX）
*   出现编码错误
*   （仅 sprintf_s）buffer 字符串长度（包括'\0'）超过 bufsz

关于返回值：

*   printf_s 与 fprintf_s 返回传输到输出流的字符数，若出现输出错误、运行时制约违规错误或编码错误则为一个负值。
*   sprintf_s 返回写入到 buffer 的字符数，不计空终止字符，若在**运行时违规返回零**，若编码错误返回负值。
*   snprintf_s 返回**返回忽略 bufsz 限制情况下本应写入的字符数**，不计空终止字符，若出现输出错误、运行时违规或编码错误则返回负值。

#### 利用 snprintf 决定包含输出的缓冲区大小

当我们不确定输出字符串的大小时，我们可以利用以零为 bufsz 和以空指针为 buffer 的 snprintf 来决定缓冲区大小。为什么呢？

当 bufsz 为零、buffer 为空指针时，根据上面的介绍，我们知道这只会**输出忽略 bufsz 限制情况下本应写入的字符数**，而且不会产生其他影响。而这个输出值，不就是我们需要的缓冲区大小值吗？

下面是一个例子：

```c
#include <stdio.h>
#include <stdlib.h>
#include <math.h>
 
int main()
{
	const char* format = "sqrt(2) = %f";
	int sz = snprintf(NULL, 0, format, sqrt(2));
	char* buf = (char*)malloc(sz + 1);	// +1 for '\0'
	snprintf(buf, sz + 1, format, sqrt(2));
	printf("%s\n", buf);
 
	return 0;
}
```