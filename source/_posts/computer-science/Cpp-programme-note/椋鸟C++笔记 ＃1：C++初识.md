---
title: 椋鸟C++笔记#1：C++初识
date: 2024-04-28 19:06:54
category:
  - [计算机科学, C++笔记]
tags:
  - C++
math: true
---

:::info
该系列为本人的学习笔记，主要由本人整理书写而成。部分内容来自教材、视频课程等，不能保证完全原创性。
:::

萌新的学习笔记，写错了恳请斧正。

### C\+\+简介

C\+\+是由 Bjarne Stroustrup 在上世纪80年代开发的一款基于C语音的编程语言。一开始是作为C语言的完善与改进，后来随着一次次标准化更新，已经脱胎换骨了。但是**C\+依旧向下兼容大部分C语言代码**。

这一篇笔记先从一些比较基础的地方讲其与C语言的区别。

### C\+\+的关键字（C\+\+98标准）

首先C语言只有32个关键字，而C\+\+98却有63个关键字，C\+\+11及之后的标准中甚至更多。

具体有哪些可以点下面的链接查看：

{% links %}
- site: C/C++参考手册
  url: https://zh.cppreference.com/w/cpp/keyword
  desc: C++关键词
  color: "#39c5bb"
  {% endlinks %}

### 命名空间

在C语言中，我们能遇到这样的情况：想要用的函数名、变量名在库中已经存在，因此不得不改名。

而在C\+\+中，我们会遇到更多的库，其中有海量的函数，再加上我们自己会写的函数，命名重复的问题就凸显出来。

因此，C\+\+就引入了**命名空间**的概念，用以==对标识符的名称进行本地化，以避免命名冲突或者名字污染==。

其原理大致可以理解为把各个函数放到各自的命名空间中，同一个命名空间内还是不能重名，但是不同命名空间之间可以有重名的函数（通过命名空间来区分）。

#### 定义命名空间

定义一个命名空间要使用关键字`namespace`，其定义方式与定义结构体类似（也同样可以嵌套）：

```cpp
int rand;
int Add(int Adder1, int Adder2)
{
    return Adder1 + Adder2;
}

namespace ExampleSpace1
{
    int rand;
    int Add(int Adder1, int Adder2)
    {
        return Adder1 + Adder2;
    }
}

namespace ExampleSpace2
{
    int rand;
    int Add(int Adder1, int Adder2)
    {
        return Adder1 + Adder2;
    }
    namespace ExampleSpace3
	{
        int rand;
        int Add(int Adder1, int Adder2)
        {
            return Adder1 + Adder2;
        }
	}
}
```

如上方所示，我们在全局、ExampleSpace1、ExampleSpace2、ExampleSpace2中的ExampleSpace3中分别定义了整型变量rand和函数Add。但是由于其所处命名空间是不同的，所以不会出现问题。

另外，在同一个工程中，可以有多个同名的命名空间，它们最后会被“合并起来”，比方说我们可以在test1.h中定义命名空间Test：

```cpp test1.h
namespace Test
{
    int a;
    int b;
}
```

然后在test2.h中继续定义命名空间Test：

```cpp test2.h
namespace Test
{
    int c;
    int d;
}
```

那么在之后使用命名空间Test时，两边的内容就相当于“合并”起来作为一个命名空间。

#### 展开（使用）命名空间

那么我们应该如何使用命名空间呢？以下面这段为例：

```cpp
int a;

namespace ExampleSpace1
{
    int a;
    int b;
}

namespace ExampleSpace2
{
    int c;
    namespace ExampleSpace3
	{
        int d;
	}
}
```

使用命名空间一般有三种方式。

##### 方式：命名空间名称加作用域限定符

两个英文冒号连在一起就是==作用域限定符（::）==。

```cpp
int main()
{
    printf("%d\n", a);	//未加限定时访问全局域的a
    printf("%d\n", ::a);	//限定为空同样访问全局域的a
    printf("%d\n", ExampleSpace1::a);	//访问ExampleSpace1中的a
    printf("%d\n", ExampleSpace2::a);	//访问ExampleSpace2中的a
    printf("%d\n", ExampleSpace2::ExampleSpace3::a);	//访问ExampleSpace2中的ExampleSpace3中的a
}
```

##### 方式：用using关键字引入命名空间内容

比方说这样就能引入ExampleSpace1中的b。

```cpp
using ExampleSpace1::b;

int main()
{
	cout << b << endl;
	return 0;
}
```

这里我们不能使用引入ExampleSpace1中的a，因为全局中已经存在一个a了，产生多义性问题。

##### 方式：用using namespace引入整个命名空间

如果某个命名空间里面有大量的内容，我们总不能一个一个去引入，所以我们可以直接引入整个命名空间：

```cpp
//注意嵌套的情况下不能直接写ExampleSpace3
using namespace ExampleSpace2::ExampleSpace3;

int main()
{
	cout << d << endl;
	return 0;
}
```

那如果我们**引入ExampleSpace2的话，能不能直接使用ExampleSpace3中的内容呢？**

答案是**否定**的，但是如果引入ExampleSpace2的话我们就可以写成这样了：

```cpp
using namespace ExampleSpace2;

int main()
{
	cout << ExampleSpace3::d << endl;
	return 0;
}
```

此时就不用写成ExampleSpace2::ExampleSpace3::d（虽然也行）。

##### 注意：命名空间展开位置

==展开（使用）命名空间是可以在作用域中进行的，不是说一定在全局中展开==，比方说：

```cpp
int main()
{
	if (true)
	{
		using namespace ExampleSpace2::ExampleSpace3;
		cout << d << endl;
	}
}
```

##### 注意：展开命名空间的实际含义

==比方说全局中的命名空间ExampleSpace1我们如果放在某作用域内展开，并不是说这个时候使用其中的变量或者函数或者别的什么就是该作用域内的东西了。这些东西依旧存在于全局，我们这里的展开只是给了一个权限使得我们能在这个作用域内定向的使用ExampleSpace1中的东西。==

这里我们可以看一个例子，比方说下面这段代码的运行结果是10，不会报错：

```cpp
int a = 0;

int main()
{
	if (true)
	{
		int a = 10;
		cout << a << endl;
	}
}
```

因为编译器会先在作用域内寻找符合的变量，这可以与全局中的变量共存。

但是如果写成这样，就会报错了：

```cpp
int a = 0;

namespace X
{
	int a = 10;
}

int main()
{
	if (true)
	{
		using namespace X;
		cout << a << endl;
	}
}
```

**这就是因为展开命名空间实际上只是给予了访问权限并不是真正的展开。编译器先在作用域内没有找到a，然后在全局中寻找，而有了命名空间X访问权限的编译器此时就在全局中发现了两个a，一个直接写在全局中，另一个写在命名空间X中。这就产生了多义性错误。**

### C\+\+的输入与输出

#### C\+\+中的标准输入输出库与标准库命名空间

我们看看C\+\+中的Hello World!程序：

```cpp
#include <iostream>
using namespace std;

int main()
{
    cout << "Hello world!" << endl;
    return 0;
}
```

第一行包含的库`<iostream>`是C\+\+的标准输入输出库，可以理解为之前C语言的`<stdio.h>`，但是内容更丰富全面。（C\+\+的库文件一般不带`.h`后缀）

第二行中的命名空间std就是C\+\+中的标准库命名空间，C\+\+所有标准库的定义都实现在这个命名空间，我们必须包含这个命名空间才能使用其中的内容。

第六行中的输出方式是C\+\+特有的IO流方式，其中`<<`是流插入运算符，与之相对的`>>`是流提取运算符。使用这种方式进行输入输出比较方便，不需要手动控制格式。其更具体的使用与原理等后面会学。

### 缺省参数

#### 缺省参数的概念

缺省参数就是在声明或者定义函数时为其参数设定一个缺省值（默认值），在调用这个函数时，如果没有指定这个参数就使用其缺省值，比方说：

```cpp
#include <iostream>

using namespace std;

void func(int a, int b = 10, int c = 20)
{
	cout << "a = " << a << ", b = " << b << ", c = " << c << endl;
}

int main()
{
	func(1);
	func(1, 2);
	func(1, 2, 3);
	return 0;
}
```

该程序输出为：

```bash
a = 1, b = 10, c = 20
a = 1, b = 2, c = 20
a = 1, b = 2, c = 3
```

#### 注意：缺省参数在参数列表中的位置

**一旦某个参数被赋予了默认值，其右侧的所有参数也必须有默认值。这是因为实参的传递是按参数列表从左到右的顺序进行的。**

也就是说，不能写出下面这种东西：

```cpp
void func(int a, int b = 10, int c)
{
	cout << "a = " << a << ", b = " << b << ", c = " << c << endl;
}
```

因为只要b有默认值，它右边的c也要有默认值才行。

#### 注意：函数有声明时缺省参数只能出现一次

如果函数同时有声明和定义，并且分开书写，那么**默认参数应只在其中之一中指定**，**通常是在函数声明中**。这是为了防止缺省参数被重定义：

```cpp
#include <iostream>

using namespace std;

void func(int a, int b = 10, int c = 20);

void func(int a, int b, int c)
{
	cout << "a = " << a << ", b = " << b << ", c = " << c << endl;
}

int main()
{
	func(1);
	func(1, 2);
	func(1, 2, 3);
	return 0;
}
```

#### 注意：缺省值必须是常量或者全局变量

### 函数重载

#### 函数重载的概念

C\+\+支持在同一个作用域中声明几个有相同函数名的函数，前提是他们能够通过参数列表区分开来。

重载函数的出现，方便了功能类似但参数类型不同的函数，举个例子：

```cpp
#include <iostream>

using namespace std;

void print(int a) 
{
	cout << "int: " << a << endl;
}

void print(double a)
{
	cout << "double: " << a << endl;
}

void print(char a)
{
	cout << "char: " << a << endl;
}

int main()
{
	print(10);
	print(10.5);
	print('A');

	return 0;
}
```

其输出为：

```bash
int: 10
double: 10.5
char: A
```

#### 函数重载实现的原理

在**C语言笔记#36**中，我们就说过C/C\+\+代码生成可执行程序要经过预处理、编译、汇编、链接几个阶段。

每一个源代码文件都要经历预处理、编译和汇编过程生成对应的`.o`文件（Linux下）或者`.obj`文件（Windows下）。最终它们与链接库通过链接器链接起来变成可执行程序。

代码中调用函数的步骤在汇编后实际上就变成了跳转到函数指令执行地址。一般来说，在多文件项目中，链接前每一个头文件是独立的，**不知道要调用的函数地址在哪。此时就会用一个“记号”来代替，而在链接阶段，链接器通过这个“记号”在符号表中寻找对应函数的地址并填进去**。

而在C语言中，这个记号仅由函数名决定，因此如果有重名函数就无法被链接器区分。

==在C\+\+中，这个记号有命名空间、函数名、参数列表多方面决定，因此能够被链接器区分开来。==

- 对于Linux环境的g\+\+编译器，这个记号由以下几个部分组成：

  `'_Z'`加上`函数名长度`加上`函数名`加上各参数类型的缩写。

  比方说函数`int Add(int a, int b);`对应的记号是：_Z3Addii

  而函数`void func(int a, float b, int* p);`对应的记号是：_Z4funcifPi

- 对于Windows环境下的名字修饰规则就比较复杂，记号由以下部分组成：

  `'?'`加上`函数名`加上`'@'`加上`命名空间（如果有嵌套用'@'隔开）`加上`'@'`加上`代表函数调用类型、参数、返回值的一串代码（具体符号含义复杂不做介绍）`加上`'@Z'`

  比方说对于函数`int func(int);`对应的记号是： ?func@@YAHH@Z

  而函数`int C::C2::func(int);`对应的记号是：?func@C2@C@@AAEHH@Z

### 引用

#### 引用的概念

在 C\+\+ 中，引用是一个非常重要的特性，它为另一个已存在的变量提供了一个别名。引用主要用于函数参数传递和返回值，使得代码更加有效且易于理解。

C\+\+使用符号 `&` 来声明引用，例如 `int& ref = var;` 这里 `ref` 就成了 `var` 的一个引用，==对引用的任何操作都是直接对原变量的操作==。

注意1：引用在声明时必须被初始化，这是与指针最大的不同之一。一旦一个引用被初始化指向一个变量，它就不能被改变指向另一个变量。

注意2：引用必须连接到一块合法的内存，不能像指针那样可以有`NULL`值。

#### 引用的操作权限

不加修饰引用的操作权限是可读可写，而权限只能缩小不能扩大，因此不能对常量进行引用：

```cpp
const int a = 10;
//int& ra = a;	//err，a为常量只读，权限不能扩大
const int& ra = a;	//ok，const修饰后ra也是只读的引用，权限不变

//int& rb = 666;	//err，666是常量只读，权限不能扩大
const int& rb = 666;	//ok

double pi = 3.14;
//int& rpi = pi;	//err，类型不同
const int& rpi = pi;	//ok，这里发生隐式的类型转换变成3
```

#### 常引用

对于上方的代码，我们注意到如果引用被const修饰，能够获得更高的灵活性。这被称为==常引用==。

- 常引用可以引用常量，这使得我们能够在不修改原始数据的前提下，安全地使用或检查其值。
- 常引用会延长临时对象的生命周期，当一个临时对象被一个常引用绑定时，该临时对象的生命周期会被延长，直到这个常引用的生命周期结束。

#### 引用在函数中的使用场景

##### 作参数

使用引用作为函数参数的主要好处是能够直接操作调用者的变量，避免了数据的复制，从而提高效率。

```cpp
void Swap(int& a, int& b)
{
    int tmp = a;
    a = b;
    b = tmp;
}
```

##### 作返回值

使用引用返回函数结果主要是为了避免返回值的拷贝，特别是当返回对象较大时。

它还允许函数返回操作结果直接对应到调用者的某个变量或对象上。

注意：==确保返回的对象在函数返回后仍然存在，要不然不要使用！==

比方说这段代码中返回的n被static修饰，在函数栈帧销毁后依旧存在，所以可用：

```cpp
int& Count()
{
    static int n = 0;
    n++;
    return n;
}
```

再比方说这段代码中返回的是array数组中的内容，不是栈帧中开辟的局部变量，所以没问题：

```cpp
#include <iostream>
using namespace std;

int& getElement(int* array, int index) 
{
    return array[index];  // 返回数组元素的引用
}

int main() 
{
    int arr[3] = {1, 2, 3};
    getElement(arr, 0) = 4;  // 通过引用修改数组元素
    cout << arr[0] << endl;  // 输出 4
    return 0;
}
```

那如果返回值是栈帧内开辟的局部变量会发生什么呢？且看下例：

```cpp
int& Add(int a, int b)
{
    int c = a + b;
    return c;
}

int main()
{
    int& ret = Add(1, 2);
    Add(3, 4);
    cout << "Add(1,2) is: " << ret << endl;
    return 0;
}
```

其输出结果并非3，而是7，为什么呢？

（现在不少编译器机制变的更加复杂，导致输出不是7而是一串随机值，但这只是一个例子）

结合我们在**C语言笔记#16**中的内容分析：

第一次调用Add函数时，1和2作为参数ab压栈，esp寄存器上移。然后创建Add函数的栈帧，esp和ebp移到Add函数的栈帧两端。变量c创建在Add函数栈帧中，而Add函数返回后esp和ebp回到main函数栈帧的两端，这片空间被释放。因此实际返回的是一片已经被释放的内存空间。

但是此时如果直接输出，其仍能输出3，因为虽然对应的空间已经被释放但是还没有被覆写。

第二次调用Add函数时，3与4压栈，esp与ebp上移，创建Add函数的栈帧。此时除了数字不同，指令地址与第一次完全一致（较新的编译器可能不是了）。因此计算结果7放在了原本第一次c的位置，也就是把原本的4覆盖了。这时再去输出ret自然就得到了Add(1, 2) = 7的荒谬结论。

#### 引用传值的优势

避免数据的拷贝，当数据占空间较大时有很大的效率优势与空间优势。

我们测试一下看看：

```cpp
#include <time.h>

struct A
{ 
	int a[10000]; 
};

A a;

A TestFunc1() { return a; }
A& TestFunc2() { return a; }

int main()
{
	// 以值作为函数的返回值类型
	size_t begin1 = clock();
	for (size_t i = 0; i < 1000000; ++i)
		TestFunc1();
	size_t end1 = clock();
	// 以引用作为函数的返回值类型
	size_t begin2 = clock();
	for (size_t i = 0; i < 1000000; ++i)
		TestFunc2();
	size_t end2 = clock();
	// 计算两个函数运算完成之后的时间
	cout << "TestFunc1 time:" << end1 - begin1 << endl;
	cout << "TestFunc2 time:" << end2 - begin2 << endl;

	return 0;
}
```

其输出：

```bash
TestFunc1 time:1610
TestFunc2 time:1
```

可见其效率差距之大。

#### 引用和指针的区别

底层上引用就是靠指针实现的，优势在于可读性更高。

我们把汇编对比一下就知道，指针实现和引用实现的汇编代码都是基本一致的。

硬要说区别的话有如下几条：

1. 引用**必须初始化**，指针不需要。
2. 引用初始化后**不能再引用其他实体**了，指针的话只要是同类型的就能随便改。
3. 有空指针**没有空引用**。
4. **sizeof关键字对指针会得到地址所占的字节数，但是对引用能得到引用类型的大小。**
5. 引用自增自减作用于被引用实体，指针则作用于自身。
6. 存在多级指针，不存在多级引用。
7. 引用比指针更安全。

### 内联函数

#### 内联函数的概念

如果程序中有一个比较小的函数被频繁调用，我们就会面临这样的问题：这个函数**太小了没什么必要去创建一个函数**，这会造成很多没必要的栈帧开销；但是这个函数确实在程序中使用的很多，**不写成函数而是拆开来写在程序里很麻烦**。

那么我们就可以使用**内联函数**解决这个问题；当然也可以使用**宏或者宏函数**解决，但是宏无法调试没有安全检查也非常容易出错，是**不推荐使用**的。

在声明/定义函数的时候，在前面加上关键字`inline`，这个函数就会变成内联函数。

编译器在处理内联函数时不会像正常函数一样写成调用函数的汇编代码，而是**直接把这个内联函数在这里展开**，比方说下面两段代码本质上没有区别：

```cpp
inline int max(int a, int b)
{
	return a > b ? a : b;
}

int main()
{
	cout << "max(10, 20) = " << max(10, 20) << endl;
	cout << "max(0, 0) = " << max(0, 0) << endl;
	cout << "max(-10, -20) = " << max(-10, -20) << endl;

	return 0;
}
```

```cpp
int main()
{
	cout << "max(10, 20) = " << (10 > 20 ? 10 : 20) << endl;
	cout << "max(0, 0) = " << (0 > 0 ? 0 : 0) << endl;
	cout << "max(-10, -20) = " << (-10 > -20 ? -10 : -20) << endl;

	return 0;
}
```

#### 内联函数的意义

内联函数是一种==以时间换空间==的做法，可能会让目标文件变的很大，但是能大大减少栈帧开销，提高运行效率。

#### 内联函数的注意事项

==注意：Visual Studio中，在debug模式下，编译器一般不会对内联函数进行展开，需在release下进行。（或在编译器配置中设置内联函数扩展的属性）==

==注意：内联函数只是向编译器提出了展开的建议，不同编译器对其实现机制可能不同。有可能写了内联编译器却不采纳，有可能没有写内联编译器却自己优化成了内联函数。==

==注意：内联函数尽量不要声明与定义分离，因为编译器处理声明时找不到函数地址（内联函数直接展开没有函数地址），然后就会报链接错误或者直接忽略内联。==

### auto关键字（C\+\+11）

#### auto关键字的概念

这里我们要说的是C\+\+11开始的auto关键字，C\+\+11之前也有auto关键字，但是含义不同。

auto关键字在C\+\+11之前用于在定义时**修饰变量为具有自动存储期的变量**，以表示这个变量会在离开当前作用域后自动归还内存。但是啊，我们如果没有单独去修饰变量为static、extern或thread_local的话，变量自然就会被创建为具有自动存储期的，正常人不会闲着没事干去加一个auto在前面。因此，auto的这个用法在C\+\+11以后已经废除。

从C\+\+11开始，**auto关键字用于替代类型名使用**。什么意思呢？

比方说==我们定义各种类型的变量，都可以直接用auto来替代类型，编译器会自动分析这里应该是什么类型==：

```cpp
#include <iostream>

using namespace std;

int main()
{
	auto a = 1;
	auto b = 3.14;
	auto c = { 1, 2 };
	auto d = "hello world";
	auto e = 'c';

	cout << typeid(a).name() << endl;
	cout << typeid(b).name() << endl;
	cout << typeid(c).name() << endl;
	cout << typeid(d).name() << endl;
	cout << typeid(e).name() << endl;

	return 0;
}
```

注意：这里的`typeid().name()`用于输出类型的名字。

其输出结果为：

```bash
int
double
class std::initializer_list<int>
char const * __ptr64
char
```

**又或者，我们可以在函数的返回值类型处写auto，编译器同样会自动分析函数的返回值类型。**

#### auto关键字的注意事项

==注意：在同一行用auto定义多个变量时，它们必须是相同类型的。==

==注意：auto声明指针类型时==`*`==可加可不加，但是声明引用类型时，必须有==`&`。

==注意：auto可以用于函数的返回值，但是不能用于函数的参数。==

==注意：auto不要直接用于声明数组。==像`auto a[] = {1, 2, 3};`这种是非法的。

### 基于范围的for循环（C\+\+11）

在之前我们如果想遍历一个数组要这样做：

```cpp
void TestForLoop()
{
	int arr[] = { 1, 2, 3, 4, 5 };
	for (int i = 0; i < sizeof(arr) / sizeof(arr[0]); i++)
	{
		arr[i] *= 2;
	}
	for (int i = 0; i < sizeof(arr) / sizeof(arr[0]); i++)
	{
		cout << arr[i] << " ";
	}
}
```

而在C\+\+11以后，可以直接写成：

```cpp
void TestForLoopWithRange()
{
	int arr[] = { 1, 2, 3, 4, 5 };
	for (auto& e : arr)
	{
		e *= 2;
	}
	for (auto e : arr)
	{
		cout << e << " ";
	}
}
```

这里for循环的括号中有冒号分割为两个部分：

- 左边是在某个范围内用于迭代的变量，一般用auto定义。如果要对范围本身的数据进行修改，就要向上面那样写成引用形式的变量。
- 右边是迭代的范围，目前我们只能接触到数组，之后还有别的迭代器。

指针空值nullptr（C\+\+11）

在我们==声明一个变量的时候，良好的习惯是在声明时进行初始化。如果声明一个指针而暂时没有指向，我们一般会将其初始化为空指针==。

在之前，我们空指针一般直接用NULL来赋值，比方说`int* p = NULL;`。

NULL实际上是一个宏，我们在`stddef.h`中能够看到其定义：

```cpp
#ifndef NULL
#ifdef __cplusplus
#define NULL	0
#else
#define NULL	((void *)0)
#endif
#endif
```

我们发现在C\+\+中，NULL被定义为0，这可能会带来一些问题，比方说下面这段代码：

```cpp
void func(int i)
{
    cout << "int" << endl;
}

void func(int* pi)
{
    cout << "int*" << endl;
}

int main()
{
    func(0);
    func(NULL);
}
```

我们发现两行输出都是int，这显然与我们的初衷相悖。

在C\+\+11以后，增加了关键字nullptr，不需要包含头文件。

nullptr默认为`((void*)0)`，**且存在到任意指针类型的隐式转换**。

我们平时使用这个关键字而不是NULL作为空指针就能解决上面的问题，能够提高代码的健壮性。