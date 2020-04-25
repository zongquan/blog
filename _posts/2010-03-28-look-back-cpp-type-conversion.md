---
layout: post
title:  "温故而知新 C++ 类型转换"
author: "Zongquan"
---

**C语言类型转换**

在C语言里用到的类型转换方式，一般都是用强制类型转换，语法：(类型说明符)(表达式)，例如： (float)a 把a转换为实型，(int)(x+y) 把x+y的结果转换为整型。
C语言这种赋值时的类型转换形式可能会使人感到不精密和不严格，因为不管表达式的值怎样，系统都自动将其转为赋值运算符左部变量的类型。

**C++类型转换**

const_cast，字面上理解就是去const属性。
static_cast，命名上理解是静态类型转换。如int转换成char。
dynamic_cast，命名上理解是动态类型转换。如子类和父类之间的多态类型转换。
reinterpret_cast，仅仅重新解释类型，但没有进行二进制的转换。 

**const_cast：**

该运算符用来修改类型的const或volatile属性。除了const 或volatile修饰之外， type_id和expression的类型是一样的。

```c++
#include <stdio.h>

using namespace std;

struct Data{
    int value;
};

int main(int argc, _TCHAR* argv[])
{
    const Data data1 = {10};
    //data1.value = 15; //error C3892: “data1”: 不能给常量赋值
    Data &data2 = const_cast<Data &>(data1);
    data2.value = 20;
    printf("data1.value = %d data2.value = %d\n",data1.value,data2.value);

    const int a = 10;
    int *b = const_cast<int*>(&a);
    *b = 20;
    printf("a = %d b == &a is %d\n",a,b == &a);

    system("pause");

    return 0;
}
```

**dynamic_cast**

有条件转换，动态类型转换，运行时类型安全检查(转换失败返回NULL)：

1. 安全的基类和子类之间转换。
2. 必须要有虚函数。
3. 相同基类不同子类之间的交叉转换。但结果是NULL。

```c++
class BaseClass {
public:
int m_iNum;
virtualvoid foo(){}; //基类必须有虚函数。保持多台特性才能使用dynamic_cast
};

class DerivedClass: public BaseClass {
public:
char*m_szName[100];
void bar(){};
};

BaseClass* pb =new DerivedClass();
DerivedClass *pd1 = static_cast<DerivedClass *>(pb); //子类->父类，静态类型转换，正确但不推荐
DerivedClass *pd2 = dynamic_cast<DerivedClass *>(pb); //子类->父类，动态类型转换，正确

BaseClass* pb2 =new BaseClass();
DerivedClass *pd21 = static_cast<DerivedClass *>(pb2); //父类->子类，静态类型转换，危险！访问子类m_szName成员越界
DerivedClass *pd22 = dynamic_cast<DerivedClass *>(pb2); //父类->子类，动态类型转换，安全的。结果是NULL
```

**static_cast：**

①用于类层次结构中基类（父类）和派生类（子类）之间指针或引用的转换。
进行上行转换（把派生类的指针或引用转换成基类表示）是安全的；
进行下行转换（把基类指针或引用转换成派生类表示）时，由于没有动态类型检查，所以是不安全的。
②用于基本数据类型之间的转换，如把int转换成char，把int转换成enum。这种转换的安全性也要开发人员来保证。
③把空指针转换成目标类型的空指针。(int* 转 long* error C2440: “static_cast”: 无法从“int *”转换为“long *”)
④把任何类型的表达式转换成void类型。

**reinterpret_cast：**

reinterpret_cast<type-id> (expression)
type-id 必须是一个指针、引用、算术类型、函数指针或者成员指针。它可以把一个指针转换成一个整数，也可以把一个整数转换成一个指针（先把一个指针转换成一个整数，再把该整数转换成原类型的指针，还可以得到原先的指针值）

主要是将原地址的值重新强制定义为新类型。

**reinterpret_cast与static_cast的类型转换比较：**

```c++
#include <stdio.h>

using namespace std;

class A
{
public:
    int m_a;
};

class B
{
public:
    int m_b;
};

class C:public A,public B
{

};



int main(int argc, _TCHAR* argv[])
{
    
    C* c = new C();
    c->m_a = 1;
    c->m_b = 2;

    A* a1 = static_cast<A*>(c);
    A* a2 = reinterpret_cast<A*>(c);

    B* b1 = static_cast<B*>(c);
    B* b2 = reinterpret_cast<B*>(c);

    printf("c = %p,a1 = %p,b1 = %p\n",c,a1,b1);
    printf("c = %p,a2 = %p,b2 = %p\n",c,a2,b2);

    printf("a1.m_a = %d, b1.m_b = %d\n",a1->m_a,b1->m_b);
    printf("a2.m_a = %d, b2.m_b = %d\n",a2->m_a,b2->m_b);

    system("pause");
    return 0;
}
```

运行结果：

```
c = 00392950,a1 = 00392950,b1 = 00392954
c = 00392950,a2 = 00392950,b2 = 00392950
a1.m_a = 1, b1.m_b = 2
a2.m_a = 1, b2.m_b = 1
```

由运行结果可以很明显地看到，使用reinterpret_cast进行强制转换的时候b2直接是从c的地址开始，将c转换成了b2，而使用static_cast的时候，b1是进行了一个sizeof(A)的一个偏移量的位移，也就是说在多重继承的情况下，将子类的对象指针转换成基类对象指针，使用static_cast才是安全的。reinterpret_cast的目的是将类型重定义，除非你非常明确地希望指向的那一块内存是希望定义到你要转换的目标，否则慎用。

**reinterpret_cast的一个使用例子：**

```c++
#include <stdio.h>

using namespace std;

typedef void(*FuncPtr)();
typedef int(*FuncPtr2)();

void fun1()
{
}

int fun2()
{
    return 10;
}


int main(int argc, _TCHAR* argv[])
{
    FuncPtr fun[10];
    fun[0] = &fun1;

    //fun[1] = &fun2; //error C2440: “=”: 无法从“int (__cdecl *)(void)”转换为“FuncPtr”
    //fun[1] = static_cast<FuncPtr>(&fun2); //error C2440: “static_cast”: 无法从“int (__cdecl *)(int)”转换为“FuncPtr”

    fun[1] = reinterpret_cast<FuncPtr>(&fun2); //用强制转换将int fun 转为 void fun 保存起来


    FuncPtr2* pfun = reinterpret_cast<FuncPtr2*>(&fun[1]); //在需要使用的时候，再转换回来

    printf("run pfun = %d\n",(*pfun)()); //打印得到 "run pfun =10"

    system("pause");
    return 0;
}
```

