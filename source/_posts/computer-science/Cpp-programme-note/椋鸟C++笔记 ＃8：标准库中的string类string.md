---
title: 椋鸟C++笔记#8：标准库中的string类
date: 2024-09-dd hh:mm:ss
category:
  - [计算机科学, C++笔记]
tags:
  - C++
  - string
math: true
---

:::info
该系列为本人的学习笔记，主要由本人整理书写而成。部分内容来自教材、视频课程等，不能保证完全原创性。
:::

萌新的学习笔记，写错了恳请斧正。

### C语言里的字符串

在学C语言的时候，我们使用过字符串，当时字符串是以'\0'结尾的字符的集合。同时，C语言标准库里还提供了一些str函数用于处理字符串。但是这种字符串的使用有很大的缺陷：

1. 关于字符串处理的库函数和字符串本身是分离的，**不符合面向对象的思想**。
2. **内存管理非常复杂**，依赖于程序员对每一个字符串进行分配和释放内存，容易出现**内存泄露或者越界访问**。
3. 很多str函数的操作并没有那么方便，想实现一些功能会有比较繁琐的操作，还要考虑内存大小的问题容易出错。
4. C语言的字符串并不方便动态扩展，需要手动重新分配内存。
5. ==虽然本质上C语言的字符串就是一个字符数组，但是这里字符串和字符数组表现出来的性质是有所不同的（详见C语言笔记#20的最后一个样例），这导致两者容易被搞混！==

### C\+\+中的string类

#### string类是什么

>C++中的`std::string`类是标准库中用于处理字符串的类。它提供了一种高效、灵活且安全的方式来操作字符序列，替代了C语言中基于`char*`的字符串。`std::string`自动管理内存，支持动态扩展，避免了手动内存分配的复杂性。它提供了丰富的成员函数和操作符重载，如拼接、查找、截取、比较等操作，极大简化了字符串的处理。此外，`std::string`还兼容STL的容器风格，支持迭代器和各种算法。

#### string类与STL的关系

首先，我们要知道string并不属于严格意义上的STL。在早期的C\+\+标准化过程中，string类是C++标准库中一个独立设计的类，设计初衷是提供比C语言中的`char*`更强大、灵活的字符串操作能力。STL和string的发展是同时的，因此STL中并没有单独做一个string，但是STL比string更早被标准化并引入C\+\+标准。

#### string类其实也是一个类模板

C++中的string本质上是基于类模板的。虽然我们通常使用string，但它实际上是basic_string类模板的一个特例化版本（`std::basic_string<char>`）。采用类模板主要是因为字符是有很多类型的，常见的是ASCII字符（char），但是还有宽字符（wchar_t）、Unicode字符（char16_t、char32_t）等其他字符需要支持。

#### string类常用接口（并不完全）

##### string类常见构造

| 构造                                                       | 解释                                                         |
| ---------------------------------------------------------- | ------------------------------------------------------------ |
| string();                                                  | 构造一个长度为0的空字符串。                                  |
| string (const string& str);                                | 拷贝构造一个字符串。                                         |
| string (const string& str, size_t pos, size_t len = npos); | 从一个已有字符串的pos位置开始，选取长度为len的子字符序列来构造一个字符串。(npos默认为-1，在size_t中即为4294967295，也就是说如果省略第三个参数就是往后有多少拷多少) |
| string (const char* s);                                    | C语言字符串转为string。                                      |
| string (const char* s, size_t n);                          | 从字符数组s中选前n个字符构造字符串。                         |
| string (size_t n, char c);                                 | 构造一个长度为n，每一个字符都是c的字符串。                   |

```cpp
#include <string>

using namespace std;

int main()
{
	string s1 = "Hello world!";
	string s2(s1);
	string s3(s1, 5, 5);
	string s4("Hello world!");
	string s5("Hello world!", 5);
	string s6(10, 'x');

	return 0;
}
```

##### string类常用容器操作

###### size与length

`size_t size() const;`

`size_t length() const;`

size和length两个函数的作用和原理完全相同，size用的更多。

size函数是为了与STL标准容器的接口保持一致而产生的，也更符合容器的抽象概念。但是length这个更符合自然语言的接口也被保留下来了，为了兼容性。

```cpp
#include <iostream>
#include <string>

using namespace std;

int main()
{
	string s("Hello, World!");
	cout << s.size() << endl;
	cout << s.length() << endl;

	return 0;
}
```

###### capacity

`size_t capacity() const;`

capacity函数返回字符串的容量。

注意：**string会默认为字符串预先分配一段最小的初始容量**，这个容量可以根据编译器实现不同而有所不同。而当容量超出先前的容量时，就会**以顺序表的方式进行容量扩展，而不是精准到每一个字节扩展**。（可见数据结构笔记#1）

比方说下面的程序在我的环境下运行输出为：15、15、15、31。

```cpp
#include <iostream>
#include <string>

using namespace std;

int main()
{
	string str1;
	cout << str1.capacity() << endl;

	string str2 = "Hello, World!";
	cout << str2.capacity() << endl;

	string str3 = "123456789012345";
	cout << str3.capacity() << endl;

	string str4 = "1234567890123456";
	cout << str4.capacity() << endl;

	return 0;
}
```

###### empty

`bool enpty() const;`

检测字符串是否为空串，如果是返回true，如果否返回false。

```cpp
#include <iostream>

using namespace std;

int main()
{
	string s1 = "a";
	cout << s1.empty() << endl;	//0

	string s2 = "";
	cout << s2.empty() << endl;	//1

	string s3;
	cout << s3.empty() << endl;	//1

	return 0;
}
```

###### clear

`void clear();`

clear函数将string中的有效字符清空，但是不会改变底层空间的大小。

```cpp
#include <iostream>

using namespace std;

int main()
{
	string s = "a";
	cout << s.size() << endl;	//1
	cout << s.capacity() << endl;	//15

	s.clear();
	cout << s.size() << endl;	//0
	cout << s.capacity() << endl;	//15
	cout << s << endl;
}
```

###### reserve

`void reserve(size_t n=0);`

reserve函数为字符串预留空间，但是不改变有效元素个数。也就是说，reserve会改变capacity，但是不会影响size。另外，reserve的参数小于string底层空间总大小时，不会进行改变。

