### 1）友元
##### （1）友元类
类并非只能拥有友元函数，还可以将类作为友元，此时友元类的所有方法都可以访问原始类的私有和保护成员

例子：让Remote类成为Tv类的一个友元
```cpp
// tv.h -- Tv and Remote classes
#ifndef TV_H_
#define TV_H_

class Tv
{
public:
    friend class Remote;   // Remote can access Tv private parts
    enum {Off, On};
    enum {MinVal,MaxVal = 20};
    enum {Antenna, Cable};
    enum {TV, DVD};

    Tv(int s = Off, int mc = 125) : state(s), volume(5),
        maxchannel(mc), channel(2), mode(Cable), input(TV) {}
    void onoff() {state = (state == On)? Off : On;}
    bool ison() const {return state == On;}
    bool volup();
    bool voldown();
    void chanup();
    void chandown();
    void set_mode() {mode = (mode == Antenna)? Cable : Antenna;}
    void set_input() {input = (input == TV)? DVD : TV;}
    void settings() const; // display all settings
private:
    int state;             // on or off
    int volume;            // assumed to be digitized
    int maxchannel;        // maximum number of channels
    int channel;           // current channel setting
    int mode;              // broadcast or cable
    int input;             // TV or DVD
};

class Remote
{
private:
    int mode;              // controls TV or DVD
public:
    Remote(int m = Tv::TV) : mode(m) {}
    bool volup(Tv & t) { return t.volup();}
    bool voldown(Tv & t) { return t.voldown();}
    void onoff(Tv & t) { t.onoff(); }
    void chanup(Tv & t) {t.chanup();}
    void chandown(Tv & t) {t.chandown();}
    void set_chan(Tv & t, int c) {t.channel = c;}
    void set_mode(Tv & t) {t.set_mode();}
    void set_input(Tv & t) {t.set_input();}
};
#endif
```
---
```cpp
// tv.cpp -- methods for the Tv class (Remote methods are inline)
#include <iostream>
#include "tv.h"

bool Tv::volup()
{
    if (volume < MaxVal)
    {
        volume++;
        return true;
    }
    else
        return false;
}

bool Tv::voldown()
{
    if (volume > MinVal)
    {
        volume--;
        return true;
    }
    else
        return false;
}

void Tv::chanup()
{
    if (channel < maxchannel)
        channel++;
    else
        channel = 1;
}

void Tv::chandown()
{
    if (channel > 1)
        channel--;
    else
        channel = maxchannel;
}

void Tv::settings() const
{
    using std::cout;
    using std::endl;
    cout << "TV is " << (state == Off? "Off" : "On") << endl;
    if (state == On)
    {
        cout << "Volume setting = " << volume << endl;
        cout << "Channel setting = " << channel << endl;
        cout << "Mode = "
            << (mode == Antenna? "antenna" : "cable") << endl;
        cout << "Input = "
            << (input == TV? "TV" : "DVD") << endl;
    }
}
```
==友元声明可以位于公有、私有、保护部分，其所在位置无关紧要==

##### （2）友元成员函数
可以选择仅让特定的类成员成为另一个类的友元，而不必让整个类成为友元，但是这样要注意各种声明和定义的顺序

例如，让 Remote() : : set_chan( ) 成为Tv类的友元的方法：在Tv类声明中将其声明为友元
```cpp
class Tv
{
	friend void Remote::set_chan(Tv& t,int c);
	...
}
```
![[Pasted image 20230806095939.png]]

##### （3）共同的友元
![[Pasted image 20230806100026.png]]
![[Pasted image 20230806100038.png]]
![[Pasted image 20230806100048.png]]

### 2）嵌套类
略
### 3）异常
##### 【1】调用abort( )
（1）函数原型位于< cstdlib >中
（2）典型实现：向标准错误流发送一个消息“程序异常终止”(abnormal program termination)，然后终止程序。它还返回一个随实现而异的值，告诉操作系统处理失败
```cpp
//error1.cpp -- using the abort() function
#include <iostream>
#include <cstdlib>
double hmean(double a, double b);
int main()
{
    double x, y, z;

    std::cout << "Enter two numbers: ";
    while (std::cin >> x >> y)
    {
        z = hmean(x,y);
        std::cout << "Harmonic mean of " << x << " and " << y
            << " is " << z << std::endl;
        std::cout << "Enter next set of numbers <q to quit>: ";
    }
    std::cout << "Bye!\n";
    return 0;
}

double hmean(double a, double b)
{
    if (a == -b)
    {
        std::cout << "untenable arguments to hmean()\n";
        std::abort();
    }
    return 2.0 * a * b / (a + b);
}

result:
Enter two numbers: 1 -1
untenable arguments to hmean()
```
ps：在 hmean( ) 程序中调用 abort( ) 函数将直接终止程序，而不是先返回到 main( )

##### 【2】返回错误码
（1）ostream类的 get(void) 成员通常返回下一个输入字符的 ASCII码；但是到达文件尾时，将返回特殊值EOF

（2）多加一个参数：使用指针参数or引用指针来将值返回给调用程序，并使用函数的返回值来指出是成功还是失败
```cpp
//error2.cpp -- returning an error code
#include <iostream>
#include <cfloat>  // (or float.h) for DBL_MAX
bool hmean(double a, double b, double * ans);
int main()
{
    double x, y, z;

    std::cout << "Enter two numbers: ";
    while (std::cin >> x >> y)
    {
        if (hmean(x,y,&z))
            std::cout << "Harmonic mean of " << x << " and " << y
                << " is " << z << std::endl;
        else
            std::cout << "One value should not be the negative "
                << "of the other - try again.\n";
        std::cout << "Enter next set of numbers <q to quit>: ";
    }
    std::cout << "Bye!\n";
    return 0;
}

bool hmean(double a, double b, double * ans)
{
    if (a == -b)
    {
        *ans = DBL_MAX;
        return false;
    }
    else
    {
        *ans = 2.0 * a * b / (a + b);
        return true;
    }
}
```
ps：第三参数可以是指针或引用。一般来说对内置的参数，倾向于采用指针，因为这样可以明显指出哪个参数用于提供答案

##### 【3】异常机制
（1）组成：
>[1]throw语句：实际上是跳转，命令程序转到另一句语句。**throw关键字**表示引发异常，随后的值(str或object)指出来异常的“人定特征”
>[2]catch模块：程序使用异常处理模块(exception handler)来捕获异常。**catch关键字**指出：当异常被引发时，程序应跳到这个位置执行
>[3]try模块：它内部存有*异常可能会被激活的代码块*，它后面跟着一个或多个catch块

（2）程序说明：
```cpp
// error3.cpp -- using an exception
#include <iostream>
double hmean(double a, double b);

int main()
{
    double x, y, z;
    std::cout << "Enter two numbers: ";
    while (std::cin >> x >> y)
    {
        try {                   // start of try block
            z = hmean(x,y);
        }                       // end of try block

        catch (const char * s)                          ----------------------------这里 const char* 与 “人定特征”string吻合
        {// start of exception handler 
            std::cout << s << std::endl;
            std::cout << "Enter a new pair of numbers: ";
            continue;
        }                       // end of handler
        std::cout << "Harmonic mean of " << x << " and " << y
            << " is " << z << std::endl;
        std::cout << "Enter next set of numbers <q to quit>: ";
    }
    std::cout << "Bye!\n";
    return 0;
}

double hmean(double a, double b)
{
    if (a == -b)
        throw "bad hmean() arguments: a = -b not allowed"; ----------------------------碰到异常时将“人定特征”设置为这个string语句
    return 2.0 * a * b / (a + b);
}
```
程序执行过程分析：
【0】程序按计划进行ing
【1】如果其中某条语句导致异常被引发，则会执行它的throw语句，这类似于执行返回语句，因为它也将终止函数的执行。但throw不是将控制权返回给调用程序，而是导致程序向函数调用的序列后退，直到找到包含try的函数
【2】找到包含它的try模块后，程序将寻找*与引发的异常类型匹配的异常处理程序*   ( 这取决于上面throw的“人定特征” )
【3】现在已找到匹配的catch模块（当异常与该处理程序匹配时，程序将执行括号中的代码）
**ps：执行完try块中的语句后，如果没有引发任何异常，则程序跳过try后面的catch块，直接执行处理程序后的第一条语句**

上例剖析：
![[Pasted image 20230806120428.png]]
![[Pasted image 20230806120441.png]]

图示解析原理：
![[Pasted image 20230806120532.png]]

（3）将类对象作为异常类型
引发异常的函数将传递一个对象，如此的优点是：可以使用不同的类型来区分不同的函数在不同的情况下引发的异常
![[Pasted image 20230806121106.png]]
![[Pasted image 20230806121315.png]]
![[Pasted image 20230806121328.png]]

举例说明：
```cpp
// exc_mean.h  -- exception classes for hmean(), gmean()
#include <iostream>
class bad_hmean
{
private:
    double v1;
    double v2;
public:
    bad_hmean(double a = 0, double b = 0) : v1(a), v2(b){}
    void mesg();
};

inline void bad_hmean::mesg()
{  
    std::cout << "hmean(" << v1 << ", " << v2 <<"): "
              << "invalid arguments: a = -b\n";
}

class bad_gmean
{
public:
    double v1;
    double v2;
    bad_gmean(double a = 0, double b = 0) : v1(a), v2(b){}
    const char * mesg();
};

inline const char * bad_gmean::mesg()
{ 
    return "gmean() arguments should be >= 0\n";
}
```
---
```cpp
//error4.cpp � using exception classes
#include <iostream>
#include <cmath> // or math.h, unix users may need -lm flag
#include "exc_mean.h"
// function prototypes
double hmean(double a, double b);
double gmean(double a, double b);
int main()
{
    using std::cout;
    using std::cin;
    using std::endl;

    double x, y, z;
    cout << "Enter two numbers: ";

    while (cin >> x >> y)
    {
        try {                  // start of try block
            z = hmean(x,y);
            cout << "Harmonic mean of " << x << " and " << y
                << " is " << z << endl;
            cout << "Geometric mean of " << x << " and " << y
                << " is " << gmean(x,y) << endl;
            cout << "Enter next set of numbers <q to quit>: ";
        }// end of try block
        
        catch (bad_hmean & bg)    // start of catch block
        {
            bg.mesg();
            cout << "Try again.\n";
            continue;
        }                  

        catch (bad_gmean & hg)
        {
            cout << hg.mesg();
            cout << "Values used: " << hg.v1 << ", "
                 << hg.v2 << endl;
            cout << "Sorry, you don't get to play any more.\n";
            break;
        } // end of catch block
    }
    cout << "Bye!\n";
    // cin.get();
    // cin.get();
    return 0;
}

double hmean(double a, double b)
{
    if (a == -b)
        throw bad_hmean(a,b);
    return 2.0 * a * b / (a + b);
}

double gmean(double a, double b)
{
    if (a < 0 || b < 0)
        throw bad_gmean(a,b);
    return std::sqrt(a * b);
}
```

##### 【4】栈解退
**（1）普通函数调用的栈过程（函数栈）：**
![[Pasted image 20230806121953.png]]

**（2）throw-catch处理机制原理：**
![[Pasted image 20230806122119.png]]

==图解：(非常直观，强烈推荐！)==
![[Pasted image 20230806122459.png]]

##### 【6】其他异常特性
>throw-catch机制类似于函数参数与函数返回的机制，但还是有区别：

（1）函数 func( ) 中的返回语句将控制权返回给调用 func( ) 的函数，但是throw语句（最终）将控制权向上返回到第一个这样的函数：包含能够捕获相应异常的try-catch组合

（2）引发异常时编译器总会创建一个临时拷贝，即使异常规范和catch模块中指定的是引用
```cpp
class problem{......};
...
void super() throw(problem)
{
	...
	if(oh_baby)
	{
		problem oops;
		throw oops;    // 这两句可以直接用：throw problem(); 来代替！
	...
}
...

try {
	super();
}

catch(problem& p)
{
	...
}
```
（1）上述的p将指向oops的副本而不是oops本身！这很好，因为函数super( ) 执行完毕后oops将不复存在！

（2）思考：既然throw语句生成的是oops的副本，为什么代码中要使用引用呢？
通常，将引用作为返回值的原因是：避免创建副本以提高效率
但是在这里：
引用有一个重要的特征：基类引用可以执行派生类的对象。假设有一组通过继承关联起来的异常类型，则在异常类型中只需列出一个基类引用，它将可以与任何派生类对象匹配！
```cpp
class bad_1 {...};
class bad_2 : public bad_1 {...};
class bad_3 : public bad_2 {...};
...

void duper()
{
	...
	if(oh_no) throw bad_1();
	if(rats) throw bad_2();
	if(drat) throw bad_3();
	...
}
...

try {
	duper();
}

catch(bad_3& be) {......}
catch(bad_2& ce) {......}
catch(bad_1& de) {......}
```
---
ps1：这里的使用要注意排列顺序！！！
【1】如果将bad_1&的catch模块放在最前面，它将捕获bad_1，bad_2 和 bad_3；
【2】通过按相反的顺序排列，bad_3异常将被bad_3&处理程序所捕获（各司其职！）

>tips：如果有一个异常类继承层次结构，应这样排列catch模块：
>将 *捕获层次结构最下面的异常类*  的catch语句放在最前面，将*捕获基类异常*的catch语句放在最后面

ps2:：对于catch模块，使用省略号来表示异常类型，从而捕获任何异常！！！
格式：
```cpp
catch(...) { //statements }
```
例子：
```cpp
try {
	duper();
}

catch(bad_3& be) { //statements }
catch(bad_2& ce) { //statements }
catch(bad_1& de) { //statements }
catch(...) { //statements }       // catch whatever is left!
```

##### 【7】exception类
c++定义了很多基于exception的异常类型
（1）stdexcept异常类：
>类别一：logic_error（典型的逻辑错误）
[1] domain_error：定义域错误
[2] invalid_argument：函数传递了一个意料之外的值
[3] length_error：没有足够的空间来执行所需的操作
[4] out_of_bounds：指示索引错误

>类别二：runtime_error（运行时错误）
[1] range_error：计算结果没有上溢或下溢，但是不在函数允许的范围内
[2] overflow_error：上溢错误（超出maximum）
[3] underflow_error：下溢错误（小于minimum）

（2）bad_alloc异常 和 new：
当无法分配请求的内存量时，new返回一个空指针
ps：如果程序在系统上运行时没有出现内存分配的问题，可以尝试提高请求分配的内存量

（3）空指针 和 new：
c++标准提供了一种在失败时返回空指针的new，形式如下：
```cpp
int* pi = new (std::nothrow) int;
int* pa = new (std::nothrow) int[666];
```


### 4）RTTI
RTTI：运行阶段类型识别 ( RunTime Type Identification)

（1）用途：
![[Pasted image 20230806134445.png]]

（2）工作原理：
【1】dynamic_cast运算符：
>如果可能的话，dynamic_cast运算符将使用*一个指向基类的指针* 来生成一个*指向派生类的指针*

语法：
```cpp
dynamic_cast<Type*> (pt);

如果指向的对象(*pt)的类型为 Type 或者是 从Type直接/间接派生而来的类型 ，则将指针pt转换为Type类型的指针-------------【指针回溯】
否则：结果是0，即空指针
```

【2】typeid运算符：
>返回一个指出对象的类型的值

【3】type_info结构：
>存储了有关特定类型的信息

==**RTTI：只适用于包含虚函数的类**==

### 5）类型转换运算符
严格限制允许的类型转换，并添加4个类型转换符，使得转换过程更规范
【1】dynamic_cast：“指针回溯”
【2】const_cast：“改变值为 const 或 volatile（不稳定的）的类型”
【3】static_cast：
【4】reinterpret_cast：

基本用不到，用到时来这里查即可（书P526~P528）