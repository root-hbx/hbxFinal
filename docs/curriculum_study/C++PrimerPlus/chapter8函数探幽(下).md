###  1）函数重载（函数多态）：

（1）功能：能让我们使用多个重名的函数，我们通过函数重载设计一系列函数——它们完成相同的工作，但使用不同的参数列表

（2）关键：函数的参数列表（也叫作：函数的特征标 function signature）
>poi：是特征标，而不是函数类型使得可以对函数进行重载！
```cpp
long gronk(int n , float m);
double gronk(int n , float m);  // not allowed!
// 这两者是互斥的！
```

（3）要点：
>【1】c++允许定义名称相同的函数，条件是特征标不同  [如果参数 数目/类型 不同，则特征标不同]
>![[Pasted image 20230728213611.png]]
>==编译器会自动根据 所采取的用法· 使用 有相应特征标 的原型==

>【2】使用被重载的函数时，需要在函数调用中提供正确的参数类型
>==point：如果某函数没有找到原型匹配，它也不会停止使用其中的某个函数，因为c++会**尝试**使用标准类型，转换强制进行匹配==
```cpp
eg1:
（1）已经有三个函数声明：
void print(const char* str , int width);  // 1)
void print(double d , int width);         // 2)
void print(const char* str);              // 3)

（2）现在要求实现：
unsigned int year = 3210;
print(year,6);

（3）分析：
没有找到完全合适的原型，2)最为接近，因此c++将year强制转化成double类型

eg2:
（1）已经有三个函数声明：
void print(int i , int width);            // 1)
void print(double d , int width);         // 2)
void print(long long l , int width);      // 3)

（2）现在要求实现：
unsigned int year = 3210;
print(year,6);

（3）分析：
有三个函数将数字作为第一个参数的原型，因此有三种转换year的方式！
这种情况下，c++不知道该咋办。
c++将拒绝这种函数调用，并将其视为错误！
```

>【3】编译器检查函数特征标时，将把类型引用和类型本身视为同一个特征标
```cpp
double cube(int x);
double cube(int &x);
参数x 在这里与原型 int x 和 int &x 都可以匹配，因此编译器无法确定究竟应当使用哪个原型！
```

>【4】要区分 const 与 非const 变量:
>==将非const值 赋给 const变量，则合法；反之是非法的！==
>![[Pasted image 20230728220502.png]]
>分析：
>dribble( ): 有两个原型，一个用于const指针，另一个用于常规指针，编译器根据实参是否是const来决定使用哪个原型！
>dabble( ): 只与带非const参数的调用匹配！
>drivel( ):  虽然表面上的原型只有含const的，但它可以与 带const 或 非const 的参数调用进行匹配！【reason：上文 黄字段】


### 2）函数模板（通用编程）：

（1）简介：函数模板是通用的函数描述，它们通过泛式来定义函数，通过将类型作为参数传递给模板，可使编译器生成该类型的函数

（2）常规模板写作格式：
```cpp
template <typename AnyType> // 也可以换成：template <class AnyType>
void hbx_swap(AnyType &a , AnyType &b)
{
	AnyType tmp;
	tmp = a;
	a = b;
	b = tmp;
}
```
>ps：==模板并不创建任何函数，而只是告诉编译器如何定义函数！==

eg1：**基本例证展示**
```cpp
// funtemp.cpp -- using a function template
#include <iostream>  // function template prototype
template <typename T>  // or class T
void Swap(T &a, T &b);

int main()
{
    using namespace std;
    int i = 10;
    int j = 20;
    cout << "i, j = " << i << ", " << j << ".\n";
    cout << "Using compiler-generated int swapper:\n";
    Swap(i,j);  // generates void Swap(int &, int &)
    cout << "Now i, j = " << i << ", " << j << ".\n";

    double x = 24.5;
    double y = 81.7;
    cout << "x, y = " << x << ", " << y << ".\n";
    cout << "Using compiler-generated double swapper:\n";
    Swap(x,y);  // generates void Swap(double &, double &)
    cout << "Now x, y = " << x << ", " << y << ".\n";
    // cin.get();
    return 0;
}

// function template definition
template <typename T>  // or class T
void Swap(T &a, T &b)
{
    T temp;   // temp a variable of type T
    temp = a;
    a = b;
    b = temp;
}
```
>ps：==函数模板只是格式变得简洁，并不能缩短可执行程序！==


eg2：**被重载的模板特征标必须不同**
```cpp
// twotemps.cpp -- using overloaded template functions
#include <iostream>
template <typename T>     // original template
void Swap(T &a, T &b);

template <typename T>     // new template
void Swap(T *a, T *b, int n);

void Show(int a[]);
const int Lim = 8;
int main()
{
    using namespace std;
    int i = 10, j = 20;
    cout << "i, j = " << i << ", " << j << ".\n";
    cout << "Using compiler-generated int swapper:\n";
    Swap(i,j);              // matches original template
    cout << "Now i, j = " << i << ", " << j << ".\n";

    int d1[Lim] = {0,7,0,4,1,7,7,6};
    int d2[Lim] = {0,7,2,0,1,9,6,9};
    cout << "Original arrays:\n";
    Show(d1);
    Show(d2);
    Swap(d1,d2,Lim);        // matches new template
    cout << "Swapped arrays:\n";
    Show(d1);
    Show(d2);
    // cin.get();
    return 0;
}

template <typename T>
void Swap(T &a, T &b)
{
    T temp;
    temp = a;
    a = b;
    b = temp;
}

template <typename T>
void Swap(T a[], T b[], int n)
{
    T temp;
    for (int i = 0; i < n; i++)
    {
        temp = a[i];
        a[i] = b[i];
        b[i] = temp;
    }
}

void Show(int a[])
{
    using namespace std;
    cout << a[0] << a[1] << "/";
    cout << a[2] << a[3] << "/";
    for (int i = 4; i < Lim; i++)
        cout << a[i];
    cout << endl;
}

这个例子里：
原模板特征标是 (T& , T&)，而新模板特征标是(T[ ], T[ ], int)
在后一个模板中，最后一个参数的类型是具体类型(int)，而不是泛式！
并非所有的模板参数都必须是模板参数类型！
```
>ps：==并非所有的模板参数都必须是模板参数类型==

（3）常规模板的局限性：
![[Pasted image 20230728232524.png]]

（4）显示具体化模板函数：
>【1】存在的目的：
>![[Pasted image 20230728233006.png]]
>
>【2】显式具体化写作格式：
```cpp
（1）非模板函数：
void swap(job& , job&)

（2）常规模板函数：
template <typename T>
void swap(T& , T&)

（3）显式具体化模板函数：
template <> void swap(job& , job&)
```

>==【3】性能比较 (优先级顺序) ：非模板函数 > 常规模板函数 > 显式具体化模板函数== 

eg：
```cpp
// twoswap.cpp -- specialization overrides a template
#include <iostream>

//1) template_func:
template <typename T>
void Swap(T &a, T &b);

struct job {
    char name[40];
    double salary;
    int floor;
};

//2) explicit specialization:
template <> void Swap<job>(job &j1, job &j2);

//3) display_func:
void Show(job &j);

int main()
{
    using namespace std;
    cout.precision(2);
    cout.setf(ios::fixed, ios::floatfield);
    
    //1) 值交换部分：
    int i = 10, j = 20;
    cout << "i, j = " << i << ", " << j << ".\n";
    cout << "Using compiler-generated int swapper:\n";
    Swap(i,j);    // generates void Swap(int &, int &)
    cout << "Now i, j = " << i << ", " << j << ".\n";
    //2) 结构交换（全换 & 部分换）
    job sue = {"Susan Yaffee", 73000.60, 7};
    job sidney = {"Sidney Taffee", 78060.72, 9};
    cout << "Before job swapping:\n";
    Show(sue);
    Show(sidney);
    
    Swap(sue, sidney); // uses void Swap(job &, job &)
    cout << "After job swapping:\n";
    Show(sue);
    Show(sidney);
    // cin.get();
    return 0;
}

template <typename T>
void Swap(T &a, T &b)    // general version
{
    T temp;
    temp = a;
    a = b;
    b = temp;
}

// swaps just the salary and floor fields of a job structure
template <> void Swap<job>(job &j1, job &j2)  // specialization
{
    double t1;
    t1 = j1.salary;
    j1.salary = j2.salary;
    j2.salary = t1;
    
    int t2;
    t2 = j1.floor;
    j1.floor = j2.floor;
    j2.floor = t2;
}

void Show(job &j)
{
    using namespace std;
    cout << j.name << ": $" << j.salary
         << " on floor " << j.floor << endl;
}

结果：
i, j = 10, 20.
Using compiler-generated int swapper:
Now i, j = 20, 10.
--------------------------------------
Before job swapping:
Susan Yaffee: $73000.60 on floor 7
Sidney Taffee: $78060.72 on floor 9
--------------------------------------
After job swapping:
Susan Yaffee: $78060.72 on floor 9
Sidney Taffee: $73000.60 on floor 7
```

>【4】实例化与具体化：
>（1）隐式实例化、显式实例化、显式具体化 统称为**具体化(specialization)**

>（2）区分显式实例化 / 显式具体化：
```cpp
[1]显式实例化：
template Func_Class Func_name<目标类型>(参数列表);     // explicit instantiation

eg: template void swap<int>(int,int);
=>  使用swap()模板生成int类型的函数定义

[2]显式具体化：
template <> Func_Class Func_name<目标类型>(参数列表);  // explicit spacialization
也可以省略<目标类型> ：
template <> Func_Class Func_name(参数列表);           // explicit spacialization
```
ps：试图在同一个文件（或转换单元）中使用同一种类型的**显式实例化**and**显式具体化**将出错！

>（3）展示使用方式：
![[Pasted image 20230729000726.png]]
对于前例中的swap()函数：
![[Pasted image 20230729000857.png]]
>
>（4）综合分析：
>![[Pasted image 20230729001036.png]]
