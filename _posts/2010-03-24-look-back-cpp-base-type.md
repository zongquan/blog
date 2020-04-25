---
layout: post
title:  "温故而知新 C++基本类型"
author: "Zongquan"
---

**C++基本类型大小：**

![](/assets/images/cpp_type_sizes.jpg)

在32位计算机中测试得到：
sizeof(bool) == 1
sizeof(char) == 1
sizeof(short) == 2
sizeof(int) == 4
sizeof(long) = 4
sizeof(float) == 4
sizeof(double) == 8

**类型枚举：**
enum

```c++
//例子： 定义一个服务器的当前状态，分别表示启动，关闭，暂停状态
enum ServerState{START, STOP, PAUSE};
```

**类型void\***

void指针可以指向任意类型的数据，亦即可用任意数据类型的指针对void指针赋值,可以用void指针来作为函数形参，这样函数就可以接受任意数据类型的指针作为参数。

```c++
int * pint;
void *pvoid;
pvoid = pint; /* 不过不能 pint= pvoid; */
void * memcpy( void *dest, const void *src, size_t len );
void * memset( void * buffer, int c, size_t num);
```

**文字量**

012132 (0开头的表示8进制)
0x1232(0x开头的表示16进制)
3U (unsigned int)
3L (long int)

按默认规定浮点数的文字量为double
2.0f
1.2e10
2.9e-3f

**limits的用法：**

```c++
The template class describes arithmetic properties of built-in numerical types. 
   
The header defines explicit specializations for the types wchar_t, bool, char, signed char, unsigned char, short, unsigned short, int, unsigned int, long, unsigned long, float, double, and long double. For these explicit specializations, the member numeric_limits::is_specialized is true, and all relevant members have meaningful values. The program can supply additional explicit specializations. Most member functions of the class describe or test possible implementations of float. 
   
For an arbitrary specialization, no members have meaningful values. A member object that does not have a meaningful value stores zero (or false) and a member function that does not return a meaningful value returns Type(0). 
 
 
这个模板类描述了内建类型的数值属性。 
   
C++标准库显式地为wchar_t, bool, char, signed char, unsigned char, short, unsigned short, int, unsigned int, long, unsigned long, float, double, and long double这些类型提供了特化。对于这些类型来说，is_specialized为true，并且所有的相关的成员（成员变量或成员函数）是有意义的。这个模板也提供其他的特化。大部分的成员函数可以用float型别来描述或测试。 
   
对于一个任意的特化，相关的成员是没有意义的。一个没有意义的对象一般用0（或者false）来表示，一个没有意义的成员函数会返回0.
```

numeric_limits的基本用法例子

```c++
#include <limits>
#include <iostream>
using namespace std;
 
int main() {
    cout << boolalpha;
 
    cout << "max(short): " << numeric_limits<short>::max() << endl;
    cout << "min(short): " << numeric_limits<short>::min() << endl;
 
    cout << "max(int): " << numeric_limits<int>::max() << endl;
    cout << "min(int): " << numeric_limits<int>::min() << endl;
 
    cout << "max(long): " << numeric_limits<long>::max() << endl;
    cout << "min(long): " << numeric_limits<long>::min() << endl;
 
    cout << endl;
 
    cout << "max(float): " << numeric_limits<float>::max() << endl;
    cout << "min(float): " << numeric_limits<float>::min() << endl;
 
    cout << "max(double): " << numeric_limits<double>::max() << endl;
    cout << "min(double): " << numeric_limits<double>::min() << endl;
 
    cout << "max(long double): " << numeric_limits<long double>::max() << endl;
    cout << "min(long double): " << numeric_limits<long double>::min() << endl;
 
    cout << endl;
 
    cout << "is_signed(char): "
        << numeric_limits<char>::is_signed << endl;
    cout << "is_specialized(string): "
        << numeric_limits<string>::is_specialized << endl;
 }
```

我的电脑运行结果

![](/assets/images/cpp_type_sizes_print.jpg)