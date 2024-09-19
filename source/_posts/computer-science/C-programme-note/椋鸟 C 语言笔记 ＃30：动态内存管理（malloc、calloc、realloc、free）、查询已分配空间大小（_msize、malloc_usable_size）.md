---
date: 2024-01-06 20:13:19
title: 椋鸟C语言笔记#30：动态内存管理（malloc、calloc、realloc、free）、查询已分配空间大小（_msize、malloc_usable_size）
category: 
  - [计算机科学, C语言笔记]
tags:
  - C
  - 内存
  - 函数
---

:::info
该系列为本人的学习笔记，主要由本人整理书写而成。部分内容来自教材、视频课程等，不能保证完全原创性。
:::

萌新的学习笔记，写错了恳请斧正。

#### 动态内存分配的意义

直接创建变量、数组等只能在栈区空间开辟固定大小的一段空间

而我们在实际使用时对空间的需求有时不会那么死板。比方说直接创建数组的话，其长度就固定了，不可调整——如果我们需要数组的大小在程序运行时才会被确定就不行了

所以 C 语言中有动态内存分配的方法，使我们能够在需要时灵活的自己申请和释放空间

#### malloc（memory allocation）

malloc 是一个开辟动态内存的函数：

```c
#include <stdlib.h>    //或者malloc.h
void *malloc( size_t size );
```

这个函数将会向内存**堆区**申请一块大小为 size（单位：字节）的**连续可用**的空间，并且返回指向这块空间的 **void\*** 类型指针，如果开辟空间**失败，则返回 NULL 指针**

- **size 为 0 是未定义的行为，其结果取决于编译器**
- **由于开辟失败可能返回 NULL，因此 malloc 的返回值一定要检查**

#### free

与 malloc 相对，free 函数用来释放我们申请的动态内存空间：

```c
#include <stdlib.h>    //或者malloc.h
void free( void *memblock );
```

- **如果参数指向的内存块不是动态开辟的，那么 free 函数的行为就是未定义的**
- **如果参数是空指针，那么这条语句将会被忽略，什么都不会发生**

#### malloc 与 free 的使用示例

```c
#include <stdio.h>
#include <stdlib.h>
int main()
{
	int num = 0;
	scanf("%d", &num);
	int* ptr = NULL;
	ptr = (int*)malloc(num * sizeof(int));
	if (ptr == NULL)	//判断ptr指针是否为空
	{
		perror("malloc");
		return 1;
	}
	for (int i = 0; i < num; i++)
		ptr[i] = i;
	for (int i = 0; i < num; i++)
		printf("%d ", ptr[i]);
	free(ptr);
	ptr = NULL;
	return 0;
}
```

注意：在使用 free 函数释放动态开辟的空间后，应**及时将指向这篇空间的指针置为空指针**（现在这些指针指向的空间是未申请的、非法的，容易造成野指针问题），这是一个良好的习惯

#### calloc（contiguous allocation）

C 语言中还有 calloc 函数，也是用来动态分配内存的：

```c
#include <stdlib.h>    //或者malloc.h
void* calloc( size_t num, size_t size );
```

这个函数的功能与 malloc 相似，在**堆区**动态开辟连续的 **num 个大小为 size 字节**的空间，但是这个函数**会把开辟出来的空间全部初始化为 0**

##### 使用示例

```c
#include <stdio.h>
#include <stdlib.h>

int main()
{
	int* p = (int*)calloc(10, sizeof(int));
	if (p == NULL)
	{
		perror("calloc");
		return 1;
	}
	for (int i = 0; i < 10; i++)
		printf("%d ", p[i]);
	free(p);
	p = NULL;
	return 0;
}
```

#### realloc（re-allocation）

有时我们要修改动态内存的大小，可以用 realloc **重新分配**动态内存空间：

```c
#include <stdlib.h>    //或者malloc.h
void *realloc( void *ptr, size_t new_size );
```

ptr 是要调整的内存的地址，new_size 是调整后的新大小，返回值是**调整之后的内存起始位置**

realloc **扩展**内存空间时有三种情况：

- **原有空间后有足够大的空间**：直接在原处追加空间，原空间数据不变（追加空间未初始化）
- **原空间后空间不足以扩展**：在堆区另外找一个合适大小的连续空间，将原空间数据复制过去（增加的空间未初始化），并将原空间释放
- **找不到足够的内存空间**：不释放原有内存块，返回空指针

realloc **收缩**内存空间时只有一种情况：将多余的部分释放

若 ptr 是 NULL，则功能与 malloc 一致

若 new_size 为 0，则行为未定义，避免这样做

##### 使用示例

由于 realloc 如果重分配失败会返回 NULL，如果我们要对 p 指针重分配，**我们就不会直接拿 p 来接受 realloc 的返回值**。因为如果返回 NULL，原空间不仅没有被释放，也没有指针指向原空间，就造成了**内存泄露**，所以我们可以这样：

```c
#include <stdio.h>
#include <stdlib.h>

int main()
{
	int* p = (int*)calloc(10, sizeof(int));
	if (p == NULL)
	{
		perror("calloc");
		return 1;
	}
	int* q = (int*)realloc(p, 20 * sizeof(int));
	if (q == NULL)
	{
		perror("realloc");
		return 1;
	}
	p = q;
	free(p);
	p = q = NULL;
	return 0;
}
```

#### 查询已分配空间大小

如果我们有一片分配好的动态内存，想知道之前到底分配了多大的空间时，我们可以使用\_msize（Windows）和 malloc_usable_size（Linux）函数来查询：

```c
#include <stdlib.h>    //或者malloc.h
size_t _msize( void *memblock );
size_t malloc_usable_size(void *_Nullable ptr);
```

这两个函数都可以返回参数指针所对应的动态内存空间实际分配的大小

**注意：这个值可能比我们开辟空间时预想的大小大（具体可以看[这个](https://www.zhihu.com/question/45775374/answer/2148195641 "这个")）**

#### 动态内存分配常见错误

##### 1. 对潜在 NULL 指针的解引用

```c
void test()
{
    int* p = (int*)malloc(INT_MAX);
    *p = 20;    //如果p的值是NULL，就会有问题
    free(p);
}
```

这里我们试图给 p 分配 INT_MAX（limits.h 中定义的值，等于整型的最大值），这很有可能分配失败返回 NULL 指针，而下面对其解引用显然就是不安全的（编译器也会报警告的）

**在动态分配内存后，一定要进行空指针判断**

##### 2. 对动态开辟空间的越界访问

```c
void test()
{
    int i = 0;
    int* p = (int*)malloc(10*sizeof(int));
    if(NULL == p)
    {
        exit(EXIT_FAILURE);
    }
    for(i=0; i<=10; i++)
    {
        *(p+i) = i;    //当i是10的时候越界访问
    }
    free(p);
}
```

其中 **exit 函数会直接终止程序并且返回其参数**

而 EXIT_FAILURE 是定义在 stdlib.h 中的错误代码，表示程序未成功执行（一般值为 1）

##### 3. 对非动态开辟内存使用 free 释放

free 函数的参数除了 NULL 只能是动态开辟的内存

##### 4. 使用 free 函数释放动态内存的一部分

如果 free 函数的参数指向的不是动态内存的起始位置，也会报错

```c
void test()
{
    int* p = (int*)malloc(100);
    p++;
    free(p);    //p不再指向动态内存的起始位置
}
```

##### 5. 多次释放同一块内存

实际上就变成了对非动态开辟内存使用 free 释放

##### 6. 动态开辟内存后忘记释放

造成 “内存泄露”：通过某个地址申请了空间，结果不还，而这个地址可能之后找不到了。这会导致这一片开辟的内存再也无法使用，如果多次内存泄露累计起来后果很严重。

**所以动态开辟的内存一定要释放**

##### 使用时常犯错误一

```c
#include <stdio.h>

void GetMemory(char* p)
{
	p = (char*)malloc(100);
}

void main()
{
	char* str = NULL;
	GetMemory(str);
	strcpy(str, "hello world");
	printf(str);
	return 0;
}
```

这段代码显然是错误的，函数传值时临时拷贝了一个形参，开辟动态内存空间后空间地址传值给了形参，而实参 str 并没有接收到动态内存空间，应该改为使用二级指针

##### 使用时常犯错误二

```c
void main()
{
	char* str = (char*)malloc(100);
	strcpy(str, "hello");
	free(str);
	if (str != NULL)
	{
		strcpy(str, "world");
		printf(str);
	}
}
```

这段代码有可能能正常的输出 world，但是实际上是错误的，str 释放后没有及时置为 NULL。后面又使用了 str，这就是典型的野指针（非法访问内存），这里可能不报错只是因为分配的那一段空间还没有被其他内容覆盖。
