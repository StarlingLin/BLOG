---
title: 椋鸟C++笔记#5：C++内存管理
date: 2024-06-11 02:15:47
category:
  - [计算机科学, C++笔记]
tags:
  - C++
  - 内存
math: true
---

:::info
该系列为本人的学习笔记，主要由本人整理书写而成。部分内容来自教材、视频课程等，不能保证完全原创性。
:::

萌新的学习笔记，写错了恳请斧正。

### C语言中的动态内存管理

C语言中，我们使用`malloc`、`calloc`、`realloc`、`free`来动态管理内存。

```c
int main()
{
    int* p = (int*)malloc(sizeof(int));	//动态开辟
    free(p);
    int* q = (int*)calloc(4, sizeof(int));	//动态开辟并赋值
    int* r = (int*)realloc(q, 10 * sizeof(int));	//重新分配空间
    free(r);
}
```

### C\+\+中的动态内存管理

C\+\+中有更简单方便的内存管理方式，那就是使用`new`、`delete`来管理内存。

#### 使用new/delete操作内置类型

==注意：如果使用new开辟空间就用delete删除，如果使用new[]开辟数组就用delete[]删除。==

1. 动态申请与删除一个内置类型变量的空间：

   ```cpp
   int* pi = new int;
   int* pf = new float;
   int* pl = new long;
   int* pb = new bool;
   
   delete pi;
   delete pf;
   delete pl;
   delete pb;
   ```

2. 动态申请与删除一个内置类型变量的空间并完成初始化：

   ```cpp
   int* pi = new int(114514);
   int* pf = new float(3.14f);
   int* pl = new long(1145141919810);
   int* pb = new bool(true);
   
   delete pi;
   delete pf;
   delete pl;
   delete pb;
   ```

3. 动态申请与删除内置类型变量**数组**的空间：

   ```cpp
   int* pi = new int[5];
   int* pf = new float[5];
   int* pl = new long[5];
   int* pb = new bool[5];
   
   delete[] pi;
   delete[] pf;
   delete[] pl;
   delete[] pb;
   ```

4. 动态申请与删除内置类型变量**数组**的空间并完成初始化：

   ```cpp
   int* pi = new int[5]{1, 2, 3, 4, 5};
   float* pf = new float[5]{1.1f, 2.2f, 3.3f, 4.4f, 5.5f};
   long* pl = new long[5]{10, 20, 30, 40, 50};
   bool* pb = new bool[5]{true, false, true, false, true};
   
   delete[] pi;
   delete[] pf;
   delete[] pl;
   delete[] pb;
   ```

#### 使用new/delete操作自定义类型

这与自定义类型区别主要在于：==使用new/delete操作自定义类型时会自动调用自定义类型的构造/析构函数。==

```cpp
class A
{
public:
    A(int a = 0)
        : m_a(a)
    {
        cout << "A" << endl;
    }
    
    ~A()
    {
        cout << "~A" << endl;
    }
    
private:
    int m_a;
};

int main()
{
    A* oA = new A;
    delete oA;
    A* oB = (A*)malloc(sizeof(A));
    free(oB);
    
    return 0;
}
```

该程序运行结果为：

```bash
A
~A
```

#### operator new(operator new[])与operator delete(operator delete[])函数

看到operator new和operator delete，很多人会以为这是new与delete的重载函数。

但是，==operator new和operator delete不能理解为是new与delete的重载函数！==

operator new与operator delete其实是系统提供的全局函数，而new和delete在底层其实就是调用了这两个函数来实现的。

==以下给出的函数定义随编译器不同有所变化！==

##### operator new函数

```cpp
void* operator new(std::size_t size) 
{
    if (size == 0)	// 如果请求的大小为0，分配至少一个字节
    {
        size = 1;
    }
    while (true)
    {
        void* p = std::malloc(size);  // 使用malloc分配内存
        if (p) {
            return p;  // 如果分配成功，返回指针
        }
        std::new_handler handler = std::get_new_handler();  // 获取当前的new_handler
        if (!handler) {
            throw std::bad_alloc();  // 如果没有设置new_handler，抛出bad_alloc异常
        }
        handler();  // 调用new_handler
    }
}
```

##### operator delete函数

```cpp
void operator delete(void* ptr) noexcept	//noexcept说明此函数不会抛出异常
{
    std::free(ptr);  // 使用free释放内存
}
```

##### operator new[]和operator delete[]

同样的，new[]和delete[]在底层调用的是operator new[]和operator delete[]函数。

```cpp
void* operator new[](std::size_t size) 
{
    if (size == 0) 
    {
        size = 1;
    }
    while (true) 
    {
        void* p = std::malloc(size);
        if (p) 
        {
            return p;
        }
        std::new_handler handler = std::get_new_handler();
        if (!handler) 
        {
            throw std::bad_alloc();
        }
        handler();
    }
}

void operator delete[](void* ptr) noexcept 
{
    std::free(ptr);
}
```

#### new与delete的实现原理

如果申请的是**内置类型**的空间，new和malloc、delete和free基本类似。

不同的是：

1. new/delete申请和释放的是单个元素的空间，new[]和delete[]申请的是**连续空间**。
2. new在申请空间失败时会抛异常；malloc会返回NULL。

对于自定义类型：

1. new的原理：
   - 调用 operator new 函数申请空间。
   - 执行构造函数。
2. delete的原理：
   - 执行析构函数。
   - 调用 operator delete 函数释放空间。
3. new[]的原理：
   - 调用 operator new[] 函数申请空间。
   - 分别执行构造函数。
   - **在这片空间前紧挨的位置申请4个字节用于保存N。**
4. delete[]的原理：
   - **往前读取4个字节，获得N。**
   - N个元素分别执行析构函数。
   - 调用 operator delete[] 函数释放空间。

#### 定位new表达式（placement-new）

我们可以直接通过指针显式的调用析构函数：

```cpp
#include <iostream>

using namespace std;

class A
{
public:
	A(int a = 0)
		: m_a(a)
	{
		cout << "A()" << this << endl;
	}

	~A()
	{
		cout << "~A()" << this << endl;
	}

private:
	int m_a;
};

int main()
{
	A* p1 = new A;
	p1->~A();
	free(p1);

	return 0;
}
```

**那我们能直接显式调用构造函数吗？答案是否定的。**

但是我们可以使用**定位new表达式**：

```cpp
#include <iostream>

using namespace std;

class A
{
public:
	A(int a = 0)
		: m_a(a)
	{
		cout << "A()" << this << endl;
	}

	~A()
	{
		cout << "~A()" << this << endl;
	}

private:
	int m_a;
};

int main()
{
	A* p1 = (A*)malloc(sizeof(A));
	new (p1) A;	//在地址p1处执行类A的构造函数
	p1->~A();
	free(p1);

	return 0;
}
```

其结构就是`new`加上`(这里放地址)`加上`类名`。

或者也可以同时初始化：

```cpp
int main()
{
	A* p1 = (A*)malloc(sizeof(A));
	new (p1) A (10);	//在地址p1处执行类A的构造函数并初始化为10
	p1->~A();
	free(p1);

	return 0;
}
```

当然我们也能显示的调用operator new(operator new[])与operator delete(operator delete[])函数：

```cpp
#include <iostream>

using namespace std;

class A
{
public:
	A(int a = 0)
		: m_a(a)
	{
		cout << "A()" << this << endl;
	}

	~A()
	{
		cout << "~A()" << this << endl;
	}

private:
	int m_a;
};

int main()
{
	A* p1 = (A*)malloc(sizeof(A));
	new (p1) A;
	p1->~A();
	free(p1);

	A* p2 = (A*)operator new(sizeof(A));
	new (p2) A(10);
	p2->~A();
	operator delete(p2);

	return 0;
}
```

### 检测内存泄露

在 Visual Stodio 中我们可以使用`_CrtDumpMemoryLeaks()`函数来检测内存泄露，如果发生内存泄露这个函数会在输出窗口给出提示。

```cpp
#include <iostream>

using namespace std;

int main()
{
	int* p = new int;

	_CrtDumpMemoryLeaks();
	return 0;
}
```

执行这段代码，我们可以在来源为调试的输出看到如下内容：

```bash
Detected memory leaks!
Dumping objects ->
{81} normal block at 0x00000221287C7400, 4 bytes long.
 Data: <    > CD CD CD CD 
Object dump complete.
```