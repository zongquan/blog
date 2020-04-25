---
layout: post
title:  "温故而知新 C++ 数组与指针"
author: "Zongquan"
---

```c++
#include <stdio.h>

using namespace std;

int main(int argc, _TCHAR* argv[])
{
    int a[5];
    int b[4] = {1,2,3,4};
    char c[] = "1234";
    int *d= new int[4];
    *d = 1;
    *(d+1) = 2;
    *(d+2) = 3;
    *(d+3) = 4;

    printf("a length = %d\n",sizeof(a));
    printf("b length = %d\n",sizeof(b));
    printf("c length = %d\n",sizeof(c));
    printf("d length = %d\n",sizeof(d));

    delete d;
    d = nullptr;

    system("pause");

    return 0;
}
```

**数组的初始化方式，可以用以上几种：**

1.直接声明，以后再对其赋值。
2.声明的时候给出初始值，用"{}"来给出初始值，如果是char类型，是可以用{"a","b","c"}或者直接用字符串赋值"abc";
3.用一个指针声明，动态分配数组的大小，例子中是定义指针的时候就从堆中申请了4个int的内存来表示数组，也是可以在要用到的时候再用new申请分配内存，并且这样的数组是需要手动去进行内存释放的，请注意下面的delete。

**运行结果：**

![](/assets/images/cpp_type_array_sizes.jpg)

在运行结果中 a和b的大小都等于 sizeof(type)*length 类型乘数组的长度，c的大小的算法也是一样的，只是因为字符数组会在最后面用一个字节长度来保存字符串的结尾字符'\0'，sizeof(d) == 4 是而并不是因为sizeof(int) == 4 ，这个4表示的仅仅是指针的大小，由于在32位的系统里是用4个字节来表示一个指针的，即无论是char* short*的指针，大小都为4。

![](/assets/images/cpp_type_array_sizes2.jpg)

如果用了一种动态申请内存的方式来表示一个字符串，请在最后一位加上'\0'，系统是不会主动地像char[N]这种方式一样，会在第N+1位填充一个'\0';

```c++
#include <stdio.h>

using namespace std;

int main(int argc, _TCHAR* argv[])
{
    char *a= new char[4];
    *a = 'a';
    *(a+1) = 'b';
    *(a+2) = 'c';
    *(a+3) = 'd'; //我们应该*(a+3) = '\0'
    //*(a+4) = '\0' //这种方式是绝对不允许的    

    printf("a strlen = %d\n",strlen(a));

    delete a;
    a = nullptr;

    system("pause");

    return 0;
}
```

运行结果：

```
a strlen = 16
```

很显示，这不是我们想要的结果，我们是希望得到的这个a的长度为4，那么我们要解决，就只能用注释里的方式，主动在最后一位填充一个'\0'

**数组与指针的关系**

```c++
int main(int argc, _TCHAR* argv[])
{
    char a[] = "abcde";
    char *p1 = a;
    char *p2 = &a[0];
    char *p3 = &a[1];

    printf("*p1 = %c, *p2 = %c\n",*p1,*p2);
    printf("*(p1+2) = %c, *(p2+2) = %c *(p3-1) = %c\n",*(p1+2),*(p2+2),*(p3-1));

    system("pause");

    return 0;
}
```

运行结果：

```c++
*p1 = a, *p2 = a
*(p1+2) = c, *(p2+2) = c *(p3+1) = a
```

指针可以方便地锁定数组的某一个位置，然后从指针当前位置进行前后游走。
数组对应着一块内存区域，而指针是指向一块内存区域。数组的地址和容量在生命期里不会改变，只有数组的内容可以改变；而指针所指向的内存区域的大小可以随时改变，而且当指针指向常量字符串时，它的内容是不可以被修改的，否则在运行时会报错。

**指针常量与常量指针**

指针常量：int *const p=&a; 这种指针是在初始化的时候对它进行赋值的，不充许之后再对p进行其它地址的赋值。
常量指针：const int* p; 这种指针允许修改它指向的地址，但是不允许修改它所指向的值。

```c++
int a=0,b=1;
    const int *p;  
    p=&a;            
    p=&b;           
    *p=2;          //不允许，不允许修改常量指针指向的值

    int *const p1=&a; 
    int *const p2;       //不允许，必须对其初始化
    p2=&b;               //不允许，p2是常量不允许作为左值
```

指针常量与常量指针多用于函数参数的限定。