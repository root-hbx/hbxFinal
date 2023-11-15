### 1）运算符重载
>[1] 格式：operator op ( argument-list )
```cpp
operator+ ()
operator* ()
operator[] ()   

ps: op 必须是有意义的c++运算符，比如 @ 就无效
```

>[2] 本质：类对象相加的函数表示法
```cpp
eg：Salesperson类重载了一个+运算符
d2 = sid + sara;
<=>
d2 = sid.operator+ (sara);
```

##### 【1】实战展示说明：
##### （1）常规操作：【注意函数 sum( ) 】
```cpp
// mytime0.h -- Time class before operator overloading
#ifndef MYTIME0_H_
#define MYTIME0_H_
class Time
{
private:
    int hours;
    int minutes;
public:
    Time();                           // 默认构造函数
    Time(int h, int m = 0);           // 构造函数
    void AddMin(int m);               
    void AddHr(int h);
    void Reset(int h = 0, int m = 0);
    const Time Sum(const Time & t) const;
    void Show() const;
};
#endif
```
-----------------------------------------------------------------------------------------------------------------
```cpp
// mytime0.cpp  -- implementing Time methods
#include <iostream>
#include "mytime0.h"
Time::Time()                   默认构造函数
{
    hours = minutes = 0;
}

Time::Time(int h, int m )      构造函数
{
    hours = h;
    minutes = m;
}

void Time::AddMin(int m)       min相加函数
{
    minutes += m;
    hours += minutes / 60;     poi！
    minutes %= 60;             poi！
}

void Time::AddHr(int h)        hour相加函数
{
    hours += h;
}

void Time::Reset(int h, int m) 重新设置函数
{
    hours = h;
    minutes = m;
}

const Time Time::Sum(const Time & t) const     求时间总量(hours,minutes)函数
{
    Time sum;
    sum.minutes = minutes + t.minutes;
    sum.hours = hours + t.hours + sum.minutes / 60;
    sum.minutes %= 60;
    return sum;
}

void Time::Show() const                       展示函数
{
    std::cout << hours << " hours, " << minutes << " minutes";
}
```
analysis：看一下Sum()函数的代码：
1）注意这里参数是引用：
目的是提高效率，如果按值传递Time对象也可以，但是传递引用的速度会更快，使用的内存会更少

2）返回值不能是引用：
因为函数**将创建一个新的Time类对象(sum)来表示另外两个Time类对象之和，返回对象(正如上述代码中)将创建对象的副本，而调用函数可以使用它**。然而，如果返回类型是Time&，则引用的是sum对象本体！但是sum对象本身是局部变量，在函数结束时就被删除，因此引用将指向一个不存在的对象。**使用返回类型Time意味着程序在删除sum之前构造它的拷贝，调用函数将得到该拷贝。**

3）const Time Time::Sum(const Time & t) const 说明：
```cpp
  const Time   Time::Sum(const Time & t)      const
 ------------  ---------------------------   ------
   返回值类型   函数头（参数）               不改变被引用对象
               要加作用域符说明属于类
```

-----------------------------------------------------------------------------------------------------------------
```cpp
// usetime0.cpp -- using the first draft of the Time class
// compile usetime0.cpp and mytime0.cpp together
#include <iostream>
#include "mytime0.h"
int main()
{
    using std::cout;
    using std::endl;
    Time planning;
    Time coding(2, 40);
    Time fixing(5, 55);
    Time total;

    cout << "planning time = ";
    planning.Show();
    cout << endl;

    cout << "coding time = ";
    coding.Show();
    cout << endl;

    cout << "fixing time = ";
    fixing.Show();
    cout << endl;

    total = coding.Sum(fixing);
    cout << "coding.Sum(fixing) = ";
    total.Show();
    cout << endl;
    // std::cin.get();
    return 0;
}
```
##### （2）基于上述sum()函数说明何为“+重载“：
只需要将sum()改成operator+ (...)即可
```cpp
// mytime1.h -- Time class before operator overloading
#ifndef MYTIME1_H_
#define MYTIME1_H_
class Time
{
private:
    int hours;
    int minutes;
public:
    ......
    Time operator+(const Time & t) const;
};
#endif
```
----------------------------------------------------------------------------------------------------------------
```cpp
......
Time Time::operator+(const Time & t) const
{
    Time sum;
    sum.minutes = minutes + t.minutes;
    sum.hours = hours + t.hours + sum.minutes / 60;
    sum.minutes %= 60;
    return sum;
}
```
----------------------------------------------------------------------------------------------------------------
```cpp
// usetime1.cpp -- using the second draft of the Time class
// compile usetime1.cpp and mytime1.cpp together
#include <iostream>
#include "mytime1.h"

int main()
{
    using std::cout;
    using std::endl;
    Time planning;
    Time coding(2, 40);
    Time fixing(5, 55);
    Time total;

    cout << "planning time = ";
    planning.Show();
    cout << endl;

    cout << "coding time = ";
    coding.Show();
    cout << endl;

    cout << "fixing time = ";
    fixing.Show();
    cout << endl;

    total = coding + fixing;               重载+ 的直接使用
    // operator notation
    cout << "coding + fixing = ";
    total.Show();
    cout << endl;

    Time morefixing(3, 28);
    cout << "more fixing time = ";
    morefixing.Show();
    cout << endl;

    total = morefixing.operator+(total);   重载+ 的原理使用
    // function notation
    cout << "morefixing.operator+(total) = ";
    total.Show();
    cout << endl;
    // std::cin.get();
    return 0;
}
```
analysis：
和sum()一样，operator+()也是由Time类对象调用的：
```cpp
total = coding.operator+ (fixing);
<=>
total = coding + fixing;
```
>ps: 在运算符表示法中，运算符左侧的对象是调用对象[隐式]，右侧的对象是作为参数被传递的对象[显式]，注意顺序！

##### （3）多对象相加的本质：
![[Pasted image 20230801095912.png]]
![[Pasted image 20230801095923.png]]

##### （4）类比上述sum()函数考察其他“重载“： - 与 *
```cpp
// mytime2.h -- Time class after operator overloading
#ifndef MYTIME2_H_
#define MYTIME2_H_
class Time
{
private:
    int hours;
    int minutes;
public:
    ...
    Time operator-(const Time & t) const;
    Time operator*(double n) const;
    ...
};
#endif
```
----------------------------------------------------------------------------------------------------------------
```cpp
Time Time::operator-(const Time & t) const
{
    Time diff;
    int tot1, tot2;
    tot1 = t.minutes + 60 * t.hours;
    tot2 = minutes + 60 * hours;
    diff.minutes = (tot2 - tot1) % 60;
    diff.hours = (tot2 - tot1) / 60;
    return diff;
}

Time Time::operator*(double mult) const
{
    Time result;
    long totalminutes = hours * mult * 60 + minutes * mult;
    result.hours = totalminutes / 60;
    result.minutes = totalminutes % 60;
    return result;
}
```
----------------------------------------------------------------------------------------------------------------
```cpp
// usetime2.cpp -- using the third draft of the Time class
// compile usetime2.cpp and mytime2.cpp together
#include <iostream>
#include "mytime2.h"
int main()
{
    using std::cout;
    using std::endl;
    Time weeding(4, 35);
    Time waxing(2, 47);
    Time total;
    Time diff;
    Time adjusted;

    cout << "weeding time = ";
    weeding.Show();
    cout << endl;

    cout << "waxing time = ";
    waxing.Show();
    cout << endl;

    cout << "total work time = ";
    total = weeding + waxing;   // use operator+()
    total.Show();
    cout << endl;

    diff = weeding - waxing;    // use operator-()
    cout << "weeding time - waxing time = ";
    diff.Show();
    cout << endl;

    adjusted = total * 1.5;      // use operator+()
    cout << "adjusted work time = ";
    adjusted.Show();
    cout << endl;
    // std::cin.get();    
    return 0;
}
```

##### 【2】重载的限制：
![[Pasted image 20230801100409.png]]


### 2）友元
【1】简介：
c++控制对类对象私有部分的访问，通过让函数成为类的友元，可以赋予该函数与类成员函数相同的访问权限

【2】问题引入：
![[Pasted image 20230801102210.png]]

【3】创建友元：
（1）将其原型放在类声明中的，并在原型声明之前加上 **关键字friend**
```cpp
class Time
{
private:
	...
public:
	......
	friend Time operator* (double m,const Time& t)
};

含义：
（1）虽然 operator* 函数是在类中声明的，但它不是成员函数，因此不能使用成员运算符来使用
（2）虽然 operator* 不是成员函数，但它与成员函数的访问权限相同
```

（2）编写函数定义
==【类内声明+类外定义】==
[1] 因为它不是成员函数，所以不要用 Time: : 限定符（限定符意思是 类自家人 属性，友元函数本质上还是外来人口）
[2] 定义中不用写friend
```cpp
Time operator* (double m,const Time& t)
{
	Time result;
	long total_min = t.hours*m*60 + t.minutes*m;
	result.hours = total_min / 60; 
	result.minutes = total_min % 60; 
	return result;
}
```

==【类内声明+类内定义】==
[1] 此时：声明 ( 也叫作：原型 ) 与定义同步，类内“声明+定义”时要**friend**
```cpp
class Time
{
private:
	...
public:
	friend Time operator* (double m,const Time& t)
	{
		return t*m;
	}
	......
};
```

【4】常用友元：重载<<运算符
（1）问题引入：
![[Pasted image 20230801105722.png]]
![[Pasted image 20230801105736.png]]

（2）重载<<的版本一：
>**要让Time类知晓使用cout，必须使用友元函数：**

常规调用是：cout << trip ;  
如果使用一个Time类成员函数来重载<<，Time对象会是第一个操作数，这意味着必须写成这种形式：trip << cout;
that's rediculous!
但是通过友元函数就可以写出合理形式：
```cpp
void operator<< (ostream& os , const Time& t)
{
	os << t.hours << "asjdhksadgh" << t.minutes << "sahk";
	//注意：返回值是void，因此没有return!
}
```
如此可以写出: cout << trip;

ps：
【1】这是Time类的友元函数，但不一定是ostream类的友元（原则上不允许修改ostream类！）
![[Pasted image 20230801110701.png]]

【2】推荐写法：传递引用！
![[Pasted image 20230801110833.png]]

（3）重载<<的版本二：
![[Pasted image 20230801111220.png]]
![[Pasted image 20230801111240.png]]

>格式：
```cpp
ostream& operator<< (ostream& os , const Time& t)
{
	os << t.hours << "asjdhksadgh" << t.minutes << "sahk";
	return os; //注意：返回值是ostream&，因此要return（cout对象）!
}
```

（4）实战分析：
```cpp
// mytime3.h -- Time class with friends
#ifndef MYTIME3_H_
#define MYTIME3_H_
#include <iostream
class Time
{
private:
    int hours;
    int minutes;
public:
    Time();
    Time(int h, int m = 0);
    void AddMin(int m);
    void AddHr(int h);
    void Reset(int h = 0, int m = 0);
    Time operator+(const Time & t) const;
    Time operator-(const Time & t) const;
    Time operator*(double n) const;
    friend Time operator*(double m, const Time & t)       // 作为内联函数
        { return t * m; }   // inline definition
    friend std::ostream & operator<<(std::ostream & os, const Time & t);
};
#endif
```
----------------------------------------------------------------------------------------------------------------
```cpp
// mytime3.cpp  -- implementing Time methods
#include "mytime3.h"
Time::Time()
{
    hours = minutes = 0;
}

Time::Time(int h, int m )
{
    hours = h;
    minutes = m;
}

void Time::AddMin(int m)
{
    minutes += m;
    hours += minutes / 60;
    minutes %= 60;
}

void Time::AddHr(int h)
{
    hours += h;
}

void Time::Reset(int h, int m)
{
    hours = h;
    minutes = m;
}

Time Time::operator+(const Time & t) const
{
    Time sum;
    sum.minutes = minutes + t.minutes;
    sum.hours = hours + t.hours + sum.minutes / 60;
    sum.minutes %= 60;
    return sum;
}

Time Time::operator-(const Time & t) const
{
    Time diff;
    int tot1, tot2;
    tot1 = t.minutes + 60 * t.hours;
    tot2 = minutes + 60 * hours;
    diff.minutes = (tot2 - tot1) % 60;
    diff.hours = (tot2 - tot1) / 60;
    return diff;
}

Time Time::operator*(double mult) const      // 友元函数作为内联函数
{
    Time result;
    long totalminutes = hours * mult * 60 + minutes * mult;
    result.hours = totalminutes / 60;
    result.minutes = totalminutes % 60;
    return result;
}

std::ostream & operator<<(std::ostream & os, const Time & t)
{
    os << t.hours << " hours, " << t.minutes << " minutes";
    return os;
}
```
----------------------------------------------------------------------------------------------------------------
```cpp
//usetime3.cpp -- using the fourth draft of the Time class
// compile usetime3.cpp and mytime3.cpp together
#include <iostream>
#include "mytime3.h"
int main()
{
    using std::cout;
    using std::endl;
    Time aida(3, 35);
    Time tosca(2, 48);
    Time temp;

    cout << "Aida and Tosca:\n";
    cout << aida<<"; " << tosca << endl;
    
    temp = aida + tosca;     // operator+()
    cout << "Aida + Tosca: " << temp << endl;
    
    temp = aida* 1.17;  // member operator*()
    cout << "Aida * 1.17: " << temp << endl;

    cout << "10.0 * Tosca: " << 10.0 * tosca << endl;
    // std::cin.get();
    return 0;
}
```

### 3）类的自动转换和强制类型转换

##### （1）c++自身特性：
>[1]   如果这两种类型兼容，则c++将自动将这个值转换成 接收变量 的类型
```cpp
long count = 8;    // 8 convert to type long
double time = 11;  // 11 convert to type double
int side = 3.33;   // 3.33 convert to type int 3
```
>[2]   c++语言不能自动转换不兼容的类型
```cpp
int* p = 10;  // invalid！

int* p = (int*) 10; // valid! 将10强制转换成int指针类型( int* )，将指针设置为10，至于是否有意义这里不关心！
```

##### （2）类中的转换：
【1】接受**一个参数**的构造函数将类型与 该参数相同的值 转换成类提供了蓝图
```cpp
Stonewt(double lbs); // double-object convert to Stonewt-object
------------------------------------------------------------------
Stonewt myCat; // create a Stonewt object
myCat = 19.6;  // use Stonewt(double) to convert 19.6 to Stonewt

使用构造函数 Stonewt(double) 来创建一个临时的Stonewt对象，并将 19.6 作为初始化值。随后，采用逐成员赋值的方式将该临时对象的内容复制到 myCat 中。这一过程叫做隐式转换（自动进行）
```

【2】**只有**接受一个参数的构造函数才能作为转换函数
```cpp
Stonewt(int s,double lbs); // not a conversion func!
```

【3】c++增添了 关键字**explicit** 用于关闭 “构造函数用作自动类型转换函数”这一特性
```cpp
explicit Stonewt(double hbx);
```
---------------------------------------------------------------------------------------------------------------------
这将关闭上述的隐式转换，但是仍然允许显式转换，即：显式强制转换
```cpp
Stonewt myCat;
myCat = 19.6;  // invalid！
myCat = Stonewt(19.6);  // OK! an explicit conversion [new form]
myCat = (Stonewt) 19.6; // OK! an explicit conversion [old form]
```

【4】注意”二义性“：
![[Pasted image 20230801154525.png]]
![[Pasted image 20230801154541.png]]

【5】当构造函数只接受一个参数时，可以使用下面的格式来初始化类对象：
```cpp
Stonewt hbx = 290;
```
这与之前说明的两类构造函数含义相同！
```cpp
法1)   Stonewt hbx(290);
法2)   Stonewt hbx = Stonewt(290);
```


##### （3）转换函数：
【1】问题引入：
![[Pasted image 20230801161858.png]]

【2】创建形式：
```cpp
operator typeName();
```
注意：
1）转换函数必须是类方法，需要通过类对象调用
2）转换函数不需要指定返回类型，因为typeName()写的就是待转换类型
3）转换函数不能有参数

eg:
```cpp
operator double();
operator int();
......
```

程序分析：
```cpp
// stonewt1.h -- revised definition for the Stonewt class
#ifndef STONEWT1_H_
#define STONEWT1_H_
class Stonewt
{
private:
    enum {Lbs_per_stn = 14};      // pounds per stone
    int stone;                    // whole stones
    double pds_left;              // fractional pounds
    double pounds;                // entire weight in pounds
public:
    Stonewt(double lbs);          // construct from double pounds
    Stonewt(int stn, double lbs); // construct from stone, lbs
    Stonewt();                    // default constructor
    ~Stonewt();
    void show_lbs() const;        // show weight in pounds format
    void show_stn() const;        // show weight in stone format

// conversion functions           (反转函数！！！)
    operator int() const;         此函数目的：实现四舍五入功能
    operator double() const;      此函数目的：直接返回该浮点数值
};
#endif
```
----------------------------------------------------------------------------------------------------------------
```cpp
// stonewt1.cpp -- Stonewt class methods + conversion functions
#include <iostream>
using std::cout;
#include "stonewt1.h"
// construct Stonewt object from double value
Stonewt::Stonewt(double lbs)
{
    stone = int (lbs) / Lbs_per_stn;    // integer division
    pds_left = int (lbs) % Lbs_per_stn + lbs - int(lbs);
    pounds = lbs;
}

// construct Stonewt object from stone, double values
Stonewt::Stonewt(int stn, double lbs)
{
    stone = stn;
    pds_left = lbs;
    pounds =  stn * Lbs_per_stn +lbs;
}

Stonewt::Stonewt()          // default constructor, wt = 0
{
    stone = pounds = pds_left = 0;
}

Stonewt::~Stonewt()         // destructor
{

}

// show weight in stones
void Stonewt::show_stn() const
{
    cout << stone << " stone, " << pds_left << " pounds\n";
}

// show weight in pounds
void Stonewt::show_lbs() const
{
    cout << pounds << " pounds\n";
}

// conversion functions
Stonewt::operator int() const      ------实现四舍五入功能
{
    return int (pounds + 0.5);
}

Stonewt::operator double()const    ------直接返回该浮点数值
{
    return pounds;
}
```
----------------------------------------------------------------------------------------------------------------
回头看看函数：
```cpp
Stonewt::operator int() const      
{
    return int (pounds + 0.5);
}

(1) Stonewt:: 说明是类方法，因此使用作用域解析符
(2) const 说明不改变被调用对象！
```

【3】当类定义了两种及以上的转换时，仍可以用显式强制类型转换来指出要使用哪个转换函数！
```cpp
long gone = (double) poppins;// 将poppins转换为一个double值；然后赋值操作：将该double值转换成long类型
long gone = int(poppins);    //                     int                       int        long
```
![[Pasted image 20230801163821.png]]
