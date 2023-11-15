### 1）动态内存分配和类：
##### （1）错误案例引入：
![[Pasted image 20230801231739.png]]
![[Pasted image 20230801231755.png]]
![[Pasted image 20230801231907.png]]
```cpp
// strngbad.cpp -- StringBad class methods
#include <cstring>                    // string.h for some
#include "strngbad.h"
using std::cout;

// initializing static class member
int StringBad::num_strings = 0;

// class methods
// construct StringBad from C string
StringBad::StringBad(const char * s)
{
    len = std::strlen(s);             // set size
    str = new char[len + 1];          // allot storage
    std::strcpy(str, s);              // initialize pointer
    
    num_strings++;                    // set object count
    cout << num_strings << ": \"" << str
         << "\" object created\n";    // For Your Information
}

StringBad::StringBad()                // default constructor
{
    len = 4;
    str = new char[4];
    std::strcpy(str, "C++");          // default string

    num_strings++;
    cout << num_strings << ": \"" << str
         << "\" default object created\n";  // FYI
}

StringBad::~StringBad()               // necessary destructor
{
    cout << "\"" << str << "\" object deleted, ";    // FYI
    --num_strings;                    // required
    cout << num_strings << " left\n"; // FYI
    delete [] str;                    // required
}

std::ostream & operator<<(std::ostream & os, const StringBad & st)
{
    os << st.str;
    return os;
}
```
![[Pasted image 20230801232420.png]]
![[Pasted image 20230801232436.png]]

>==圈画处是程序中的亮点，值得学习借鉴！但是程序本身存在严重的错误！==

##### （2）案例错误原因简析：
![[Pasted image 20230801232812.png]]

##### （3）特殊成员函数：
【1】默认构造函数：
1）如果没有提供任何构造函数，c++将创建默认构造函数
```cpp
假如定义了一个K类，但是没有提供任何构造函数，则编译器将提供下述默认构造函数：
K::K() { }
这提供了一个不接受任何参数，也不执行任何操作的构造函数
K hbx;
默认构造函数使得 hbx 类似于一个常规的自动变量，也就是说：它的值在初始化时未知！
```
2）如果定义了构造函数，c++将不会定义默认构造函数
```cpp
如果希望在创建对象时不显式地对它进行初始化，则必须显式地定义默认构造函数
K::K()
{
	ct = 0;
	......
}
```
3）带参数的构造函数也可以是默认构造函数，只要所有函数都有默认值
4）只能有一个默认构造函数
```cpp
【1】Kunk() {ct = 0;}
【2】Kunk(int n = 0) {ct = n;}
上述写法不合法！
比如： Kunk ikun;  // 可以与声明1与2都匹配！
```

【2】复制构造函数：
（1）理论上的出现时机：
>用于将一个对象复制到**新创建的对象**中。也就是说：它用于**初始化过程**中(包括按值传递参数)，而==不是常规的赋值==过程中

（2）类的复制构造函数：
```cpp
Class_name (const Class_name &) // 它接受一个指向类对象的常量引用作为参数
```
比如：StringBad类的复制构造函数的原型是：StringBad (const StringBad &);

（3）何时调用复制构造函数：
>**[1] 新建一个对象并将其初始化为同类现有对象时，复制构造函数将被调用**
>这其中最常见的情况是：新对象显式地初始化为现有的对象
```cpp
假设motto是一个StringBad对象，则下面四种声明都将调用复制构造函数：
StringBad ditto(motto);           //直接初始化
StringBad metto = motto;          //可能使用复制构造函数直接创建metto；也可能使用复制构造函数生成一个临时对象，然后将临时对象的内容赋给metto
StringBad also = StringBad(motto);//可能使用复制构造函数直接创建also；也可能使用复制构造函数生成一个临时对象，然后将临时对象的内容赋给also
StringBad *p = new StringBad(motto); //使用motto初始化一个匿名对象，并将新对象的地址赋给 p指针
```
>
>[2] **每当程序生成了对象副本时，编译器都将使用复制构造函数**
>具体而言：当函数按值传递对象or函数返回对象时，都将使用复制构造函数（按值传递意味着创建原始变量的一个副本，编译器生成临时对象时也将使用复制构造函数）
>ps：由于按值传递对象将调用复制构造函数，因此应当按引用传递对象，这样提高效率！

（4）默认复制构造函数的功能：
>逐个复制非静态成员（成员复制也叫作：浅复制），复制的是成员的值
>ps：静态成员不受影响，因为它们属于整个类！
![[Pasted image 20230802101856.png]]

##### 4）回望：之前的复制构造函数究竟错在哪里？
![[Pasted image 20230802102158.png]]
![[Pasted image 20230802102211.png]]

解决方案：
进行==深度复制（deep copy）==：复制构造函数应当复制字符串并将一个==副本的地址==赋给str成员，而**不是复制字符串的地址**。这样每个对象都有自己的字符串，而不是引用另一个对象的字符串。调用析构函数时都将会释放不同的字符串，而不会试图去释放已经被释放的字符串！
```cpp
StringBad::StringBad(const StringBad & st)
{
	num_string++;
	len = st.len;
	str = new char [len + 1];
	std::strcpy(str , st.str);
}
```
>tips：如果类中包含了使用new初始化的指针成员，应当定义一个复制构造函数，以复制指向的数据，而不是指针，这叫深度复制；

图解：
![[Pasted image 20230802103305.png]]

##### 5）回顾之前的函数还有其它问题：赋值运算符
（1）原型：
```cpp
Class_name& Class_name::operator= (const Class_name &)  // 它接受并返回一个指向类对象的引用
```
（2）功能：
[1] 将已有的对象赋给另一个对象时，使用重载的赋值运算符
```cpp
StringBad headline1("nasdjaksl");
...
StringBad knot;
knot = headline1;
```
[2] 初始化对象时，并不一定会使用赋值运算符
```cpp
比如：
StringBad metto = knot;
metto是一个新创建的对象，被初始化为knot的值，因此使用复制构造函数
```
[3] 与复制构造函数类似，赋值运算符的隐式实现也会对成员逐个复制！

（3）解决赋值运算符的方案：
>深度复制！

具体要求：
【1】由于目标对象可能引用了**以前分配的数据**，所以函数应使用 delete[ ] 来**释放**这些数据
【2】函数应当**避免将对象赋给自身**；否则给对象重新复制前，释放内存可能会误删对象的数据
【3】函数**返回**一个**指向调用对象的引用**
![[Pasted image 20230802104526.png]]

```cpp
StringBad& StringBad::operator=(const StringBad& st)
{
	if(this == &st) return *this;  // 【2】
	delete [] str;                 // 【1】free old string
	len = st.len;                 
	str = new char[len+1];         // get space for new string
	std::strcpy(str,st.str);       // copy the string
	return *this;                  // return reference to invoking object
}

//1) code首先进行自我检查，看目标地址与接受对象地址(this)是否相同
//2) 如果地址不同，函数将释放str指向的内存：这是因为稍后将把一个新字符串的地址赋给str，如果不先使用delete,则上述字符串将遗留在内存中
//3) 为新字符串分配足够的内存空间 + 字符串复制进新的内存单元
//4）return *this;
```


### 2）改进后的新String类：
【1】程序整体：
```cpp
// string1.h -- fixed and augmented string class definition
#ifndef STRING1_H_
#define STRING1_H_
#include <iostream>
using std::ostream;
using std::istream;
class String
{
private:
    char * str;             // pointer to string
    int len;                // length of string
    static int num_strings; // number of objects
    static const int CINLIM = 80;  // cin input limit
public:
// constructors and other methods                                 构造函数              
    String(const char * s); // constructor                        非默认构造函数
    String();               // default constructor                默认构造函数
    String(const String &); // copy constructor                   复制构造函数
    ~String();              // destructor
    int length () const { return len; }
// overloaded operator methods    
    String & operator=(const String &);                           //重载赋值运算符（对象风格）
    String & operator=(const char *);                             //重载赋值运算符（C风格）
    char & operator[](int i);                                     //中括号符（可改正）
    const char & operator[](int i) const;                         //中括号符（仅查看）
// overloaded operator friends
    friend bool operator<(const String &st, const String &st2);    //重载比较符号
    friend bool operator>(const String &st1, const String &st2);   //重载比较符号
    friend bool operator==(const String &st, const String &st2);   //重载比较符号
    friend ostream & operator<<(ostream & os, const String & st);  //重载输入输出流
    friend istream & operator>>(istream & is, String & st);        //重载输入输出流
// static function
    static int HowMany();
};
#endif
```
---------------------------------------------------------------------------------------------------------------------
```cpp
// string1.cpp -- String class methods
#include <cstring>                 // string.h for some
#include "string1.h"               // includes <iostream>
using std::cin;
using std::cout;
// initializing static class member
int String::num_strings = 0;
// static method
int String::HowMany()
{
    return num_strings;
}

// class methods
String::String(const char * s)     // construct String from C string         初始化构造函数
{
    len = std::strlen(s);          // set size
    str = new char[len + 1];       // allot storage
    std::strcpy(str, s);           // initialize pointer
    num_strings++;                 // set object count
}

String::String()                   // default constructor
{
    len = 4;
    str = new char[1];
    str[0] = '\0';                 // default string
    num_strings++;
}

String::String(const String & st)                                         // 复制构造函数
{
    num_strings++;             // handle static member update
    len = st.len;              // same length
    str = new char [len + 1];  // allot space
    std::strcpy(str, st.str);  // copy string to new location
}

String::~String()                     // necessary destructor
{
    --num_strings;                    // required
    delete [] str;                    // required
}

// overloaded operator methods  

    // assign a String to a String
String & String::operator=(const String & st)
{
    if (this == &st)
        return *this;
    delete [] str;
    len = st.len;
    str = new char[len + 1];
    std::strcpy(str, st.str);
    return *this;
}
    // assign a C string to a String
String & String::operator=(const char * s)
{
    delete [] str;
    len = std::strlen(s);
    str = new char[len + 1];
    std::strcpy(str, s);
    return *this;
}
    // read-write char access for non-const String
char & String::operator[](int i)
{
    return str[i];
}
    // read-only char access for const String
const char & String::operator[](int i) const
{
    return str[i];
}

// overloaded operator friends
bool operator<(const String &st1, const String &st2)
{
    return (std::strcmp(st1.str, st2.str) < 0);
}

bool operator>(const String &st1, const String &st2)
{
    return st2 < st1;
}

bool operator==(const String &st1, const String &st2)
{
    return (std::strcmp(st1.str, st2.str) == 0);
}
    // simple String output
ostream & operator<<(ostream & os, const String & st)
{
    os << st.str;
    return os;
}
    // quick and dirty String input
istream & operator>>(istream & is, String & st)
{
    char temp[String::CINLIM];
    is.get(temp, String::CINLIM);
    if (is)
        st = temp;
    while (is && is.get() != '\n')
        continue;
    return is;
}
```

【2】程序分析：
![[Pasted image 20230802110522.png]]
![[Pasted image 20230802110646.png]]
![[Pasted image 20230802111015.png]]
![[Pasted image 20230802111034.png]]
![[Pasted image 20230802111144.png]]

【3】重难点赏析：
（1）初始化构造函数 and 复制构造函数：
```cpp
初始化构造函数：
String::String(const char * s)     // construct String from C string         初始化构造函数
{
    len = std::strlen(s);          // set size
    str = new char[len + 1];       // allot storage
    std::strcpy(str, s);           // initialize pointer
    num_strings++;                 // set object count
}

----------------------------------------------------------------------------------------------------

复制构造函数：
String::String(const String & st)                                         // 复制构造函数
{
    num_strings++;             // handle static member update
    len = st.len;              // same length
    str = new char [len + 1];  // allot space
    std::strcpy(str, st.str);  // copy string to new location
}
```
从代码角度分析：为什么初始化创造的是“地址传递”，而复制构造函数创造的是“副本地址传递”
```cpp
初始化构造函数：

String::String(const char * s)   （1）
与
std::strcpy(str, s);             （2）

将字符串的地址指针(s) 直接copy给str；str的指针指向的就是字符串的地址
因此二者指向的是同一个地址【会导致double free】
```
---------------------------------------------------------------------------------------------------------------------
```cpp
复制构造函数：
String::String(const String & st)  （1）         
与
std::strcpy(str, st.str);          （2）

（1） 将该类对象先产生副本
（2）副本将地址指针copy给str；str的指针指向的就是字符串的副本的地址
因此二者指向的是分别的地址【不会导致double free】
```

（2）“对象风格”和“C风格”的赋值符重载：
```cpp
    // assign a String to a String
String & String::operator=(const String & st)
{
    if (this == &st)
        return *this;
    delete [] str;
    len = st.len;
    str = new char[len + 1];
    std::strcpy(str, st.str);
    return *this;
}
---------------------------------------------------------
    // assign a C string to a String
String & String::operator=(const char * s)
{
    delete [] str;
    len = std::strlen(s);
    str = new char[len + 1];
    std::strcpy(str, s);
    return *this;
}
```
本质上这两种写法没有任何区别，只是C风格写法的效率高一点

（3）中括号字符组的“可修改”与“仅查看”写法对比：
```cpp
char& String::operator[] (int i)                                    可修改
{
	return str[i];
}

const char& String::operator[] (int i) const                        仅查看
{
	return str[i];
}
```

##### 3）构造函数中使用new时应注意的问题：
（1）基础注意事项：
![[Pasted image 20230802113716.png]]
![[Pasted image 20230802113731.png]]

（2）高级注意事项：
【1】深度复制：复制构造函数
```cpp
String::String(const String & st)
{
	num_string++;
	len = st.len;
	str = new char [len + 1];
	std::strcpy(str , st.str);
}
```

【2】深度复制：赋值运算符
```cpp
String& String::operator= (const String & st)
{
	if(this == &st) return *this;
	delete str;
	len = st.len;
	str = new char [len+1]；
	std::strcpy(str , st.str);
	return *this;
}
```

##### 4）有关返回对象的说明：
![[Pasted image 20230802115048.png]]
![[Pasted image 20230802115108.png]]
![[Pasted image 20230802115124.png]]

##### 5）使用指向对象的指针：
```cpp
String* fab = new String( sayings[choice] );
```
【1】这里指针fab指向new创建的未被命名对象
【2】使用对象 sayings[choice] 来初始化新的String对象，这将调用复制构造函数 

>ps：在下列情况下将调用析构函数
>1）如果对象是动态变量，则当执行完定义该对象的程序块时，将调用该对象的析构函数
>2）如果对象是静态变量，则在程序结束时调用该对象的析构函数

小结：指针和对象
（1）常规表示法：
```cpp
String* gla;
```
（2）可以将指针初始化为指向已知的对象
```cpp
String* fir = &sayings[10];
```
（3）可以使用new来初始化指针，这将创建一个新的对象
```cpp
String* fab = new String( sayings[choice] ); 
```
（4）对类使用new将调用相应的类构造函数，来初始化新创建的对象
```cpp
String* glp = new String;
String* op = new String("my my my");
String* rp = new String( sayings[choice] );
```
（5）可以使用 -> 来通过指针访问类方法
```cpp
if(sayings[i].length() > shortest->length()) 
{
	...
}
```
（6）解引用可以使用