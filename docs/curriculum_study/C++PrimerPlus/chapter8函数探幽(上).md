### 1）内联函数的定义与创作目的：
（1）基本概念：
![[Pasted image 20230728093518.png]]

（2）图解：
![[Pasted image 20230728093541.png]]

（3）通用格式：下面二选一即可
```cpp
（1）函数声明前加上 关键字 inline;
（2）函数定义前加上 关键字 inline;
```

ps:  函数过大or需要函数递归时，不能将其作为内联函数（reason: 内联函数不能递归！）

（4）代码演示：
```cpp
// inline.cpp -- using an inline function

#include <iostream>
using namespace std;
inline double square(double x) { return x * x; }  // an inline function definition
int main()
{
    double a, b;
    double c = 13.0;
    
    a = square(5.0);
    b = square(4.5 + 7.5);   // can pass expressions

    cout << "a = " << a << ", b = " << b << "\n";
    cout << "c = " << c;
    cout << ", c squared = " << square(c++) << "\n";
    cout << "Now c = " << c << "\n";
    // cin.get();
    return 0;  
}

结果：
a = 25, b = 144
c = 13, c squared = 169
Now c = 14
```
>（1）内联函数与常规函数一样，按值传递参数
>（2）如果参数为表达式，则传递表达式的值
>（3）注意这一步：b = square(4.5 + 7.5);   // can pass expressions
>可以给square()传递int或long值，将值传递给函数前，程序将自动把这个值强制转换double类型!

（5）拓展：内联与宏
>内联函数按值传递;
>宏是通过文本替换实现的，不能按值传递；
```cpp
宏定义：
#define square(x) x*x
```
![[Pasted image 20230728100214.png]]
![[Pasted image 20230728100233.png]]


### 2）引用变量：

##### （1）创建引用变量：
[1]引用变量的主要用途是作为函数的形参；通过将引用变量用作参数，函数将使用其原始数据，而不是其副本

[2]如何创建引用变量：
```cpp
int rats;
int & rodents = rats;

&不是地址运算符，而是类型标志符的一部分，int& 指的是：指向int的引用
上述声明将rats与rodents互换，它们指向相同的值与内存单元
```

```cpp
// firstref.cpp -- defining and using a reference
#include <iostream>
using namespace std;
int main()
{
    int rats = 101;
    int & rodents = rats;   // rodents is a reference
    
    cout << "rats = " << rats;
    cout << ", rodents = " << rodents << endl;
    
    rodents++;
    cout << "rats = " << rats;
    cout << ", rodents = " << rodents << endl;

    cout << "rats address = " << &rats;
    cout << ", rodents address = " << &rodents << endl;
    // cin.get();
    return 0;
}

结果：
rats = 101, rodents = 101
rats = 102, rodents = 102
rats address = 0x61fe14, rodents address = 0x61fe14
```
分析得知：
【1】一旦建立引用关系，二者的值和地址都相同[耦合在一起]
【2】其中一者发生改变会随之自动改变另一个

ps：指针和引用的异同：
>[1]必须在声明引用时将其初始化；而不能像指针那样，先声明，后赋值
```cpp
int rat;
int & rodents;
rodents = rat;   //no！you cannot do this ！ 
```
>[2]引用更接近const指针，一旦关联建立，就一直效忠于它
```cpp
int & rodents = rats;
int * const pr = &rats;

其中 引用rodents 与 表达式*pr 扮演的角色相同！
```

[3]综合分析：
```cpp
// secref.cpp -- defining and using a reference
#include <iostream>
int main()
{
    using namespace std;
    int rats = 101;
    int & rodents = rats;   // rodents is a reference

    cout << "rats = " << rats;
    cout << ", rodents = " << rodents << endl;

    cout << "rats address = " << &rats;
    cout << ", rodents address = " << &rodents << endl;

    int bunnies = 50;
    rodents = bunnies;       // can we change the reference?
    
    cout << "bunnies = " << bunnies;
    cout << ", rats = " << rats;
    cout << ", rodents = " << rodents << endl;

    cout << "bunnies address = " << &bunnies;
    cout << ", rodents address = " << &rodents << endl;
    // cin.get();
    return 0;
}

结果：
rats = 101, rodents = 101
rats address = 0x61fe14, rodents address = 0x61fe14
bunnies = 50, rats = 50, rodents = 50
bunnies address = 0x61fe10, rodents address = 0x61fe14
```
很有意思的现象：
![[Pasted image 20230728103245.png]]
![[Pasted image 20230728103325.png]]

##### （2）将引用作为函数参数：
[1] 按引用传递，允许被调用函数能够访问调用函数中的变量
[2] 按值传递，被调用函数使用 调用程序的值的拷贝
![[Pasted image 20230728160204.png]]
>分析问题：交换两个变量的值
>（1）交换函数必须能够修改调用程序的变量的值，这意味着按值传递变量将不管用，因为函数将交换原始数据变量的 ==副本== 的内容
>key1: 传递引用时，函数使用的是原始数据
>key2: 传递指针访问原始数据

示例：
```cpp
// swaps.cpp -- swapping with references and with pointers
#include <iostream>
void swapr(int & a, int & b);   // a, b are aliases for ints
void swapp(int * p, int * q);   // p, q are addresses of ints
void swapv(int a, int b);       // a, b are new variables
int main()
{
    using namespace std;
    int wallet1 = 300;
    int wallet2 = 350;

    cout << "wallet1 = $" << wallet1;
    cout << " wallet2 = $" << wallet2 << endl;

// （1）引用传递：
    cout << "Using references to swap contents:\n";
    swapr(wallet1, wallet2);   // pass variables
    cout << "wallet1 = $" << wallet1;
    cout << " wallet2 = $" << wallet2 << endl;
// （2）指针传递：
    cout << "Using pointers to swap contents again:\n";
    swapp(&wallet1, &wallet2); // pass addresses of variables
    cout << "wallet1 = $" << wallet1;
    cout << " wallet2 = $" << wallet2 << endl;
// （3）值传递：
    cout << "Trying to use passing by value:\n";
    swapv(wallet1, wallet2);   // pass values of variables
    cout << "wallet1 = $" << wallet1;
    cout << " wallet2 = $" << wallet2 << endl;
    // cin.get();
    return 0;
}

void swapr(int & a, int & b)    // use references
{
    int temp;
    temp = a;       // use a, b for values of variables
    a = b;
    b = temp;
}

void swapp(int * p, int * q)    // use pointers
{
    int temp;
    temp = *p;      // use *p, *q for values of variables
    *p = *q;
    *q = temp;
}

void swapv(int a, int b)        // try using values
{
    int temp;
    temp = a;      // use a, b for values of variables
    a = b;
    b = temp;
}
```

![[Pasted image 20230728161741.png]]

##### （3）引用的属性与特别之处：
>[1]问题引入：
```cpp
// cubes.cpp -- regular and reference arguments
#include <iostream>
double cube(double a);
double refcube(double &ra);
int main ()
{
    using namespace std;
    double x = 3.0;

    cout << cube(x);
    cout << " = cube of " << x << endl;

    cout << refcube(x);
    cout << " = cube of " << x << endl;
    // cin.get();
    return 0;
}

double cube(double a)
{
    a *= a * a;
    return a;
}

double refcube(double &ra)
{
    ra *= ra * ra;
    return ra;
}

结果：
27 = cube of 3
27 = cube of 27
```
>**refcube()函数修改了main()中的x值，而cube()没有**
>（1）变量a位于cube()中，它被初始化为x的值，但是修改a并不会影响x
>（2）由于refcube()使用了引用参数，因此修改ra实际上就是修改x
>==修正方法：double refcube (const double &ra) ;==

>[2]临时变量、引用参数、const：
> （1）当实参与引用参数不匹配时，c++将生成临时变量【iff：引用参数是const】
> （2）何时创建临时变量？
> 【1】实参的类型正确，但不是左值；
> 【2】实参的类型不正确，但是可以被转换成正确的类型；
> ![[Pasted image 20230728165448.png]]
> 解释：
> （1）参数side、len[2]、rd、****pd*** 都是有名称的，且是double类型的数据对象，因此可以为其创建引用
> （2）edge虽然是变量，但是类型却不正确，double引用不能指向long
> （3）7.0和side+1.0类型正确，但是没有名称
> （4）因此对于（2）和（3），编译器都将生成一个临时的匿名变量，并让ra指向它；这些临时变量只在函数调用期间存在，此后编译器便可以将其随意删除！

![[Pasted image 20230728170731.png]]

==ps：左值与右值==
（1）左值：
- 变量名、函数名以及数据成员名
- 返回左值引用的函数调用
- 由赋值运算符或复合赋值运算符连接的表达式，如(a=b, a-=b等)
- 解引用表达式*ptr
- 前置自增和自减表达式(++a, ++b)
- 成员访问（点）运算符的结果
- 由指针访问成员（ `->` ）运算符的结果
- 下标运算符的结果(`[]`)
- 字符串字面值("abc")

（2）右值：
- 字面值(字符串字面值除外)，例如1，'a', true等
- 返回值为非引用的函数调用或操作符重载，例如：str.substr(1, 2), str1 + str2, or it++
- 后置自增和自减表达式(a++, a--)
- 算术表达式
- 逻辑表达式
- 比较表达式
- 取地址表达式
- lambda表达式

##### （4）将引用用于结构：
![[Pasted image 20230728171115.png]]

问题分析：
```cpp
//strc_ref.cpp -- using structure references
#include <iostream>
#include <string>
struct free_throws{
    std::string name;
    int made;
    int attempts;
    float percent;
};

//函数声明部分：
void display(const free_throws & ft);
void set_pc(free_throws & ft);
free_throws & accumulate(free_throws &target, const free_throws &source);

int main()
{
    free_throws one = {"Ifelsa Branch", 13, 14};
    free_throws two = {"Andor Knott", 10, 16};
    free_throws three = {"Minnie Max", 7, 9};
    free_throws four = {"Whily Looper", 5, 9};
    free_throws five = {"Long Long", 6, 14};
    free_throws team = {"Throwgoods", 0, 0};
    free_throws dup;

    set_pc(one);
    display(one);
    accumulate(team, one);
    display(team);
// use return value as argument
    display(accumulate(team, two));
    accumulate(accumulate(team, three), four);
    display(team);
// use return value in assignment
    dup = accumulate(team,five);
    std::cout << "Displaying team:\n";
    display(team);
    std::cout << "Displaying dup after assignment:\n";
    display(dup);
    set_pc(four);
// ill-advised assignment
    accumulate(dup,five) = four;
    std::cout << "Displaying dup after ill-advised assignment:\n";
    display(dup);
    // std::cin.get();
    return 0;
}

void display(const free_throws & ft)
{
    using std::cout;
    cout << "Name: " << ft.name << '\n';
    cout << "  Made: " << ft.made << '\t';
    cout << "Attempts: " << ft.attempts << '\t';
    cout << "Percent: " << ft.percent << '\n';
}

void set_pc(free_throws & ft)
{
    if (ft.attempts != 0)
        ft.percent = 100.0f *float(ft.made)/float(ft.attempts);
    else
        ft.percent = 0;
}

free_throws & accumulate(free_throws & target, const free_throws & source)
{
    target.attempts += source.attempts;
    target.made += source.made;
    set_pc(target);
    return target;
}

结果：
Name: Ifelsa Branch
  Made: 13      Attempts: 14    Percent: 92.8571
Name: Throwgoods
  Made: 13      Attempts: 14    Percent: 92.8571
Name: Throwgoods
  Made: 23      Attempts: 30    Percent: 76.6667
Name: Throwgoods
  Made: 35      Attempts: 48    Percent: 72.9167
Displaying team:
Name: Throwgoods
  Made: 41      Attempts: 62    Percent: 66.129
Displaying dup after assignment:
Name: Throwgoods
  Made: 41      Attempts: 62    Percent: 66.129
Displaying dup after ill-advised assignment:
Name: Whily Looper
  Made: 5       Attempts: 9     Percent: 55.5556
```

[1]程序说明：
![[Pasted image 20230728175029.png]]

[2]为什么返回引用：
![[Pasted image 20230728175228.png]]

[3]返回引用需要注意：
![[Pasted image 20230728175411.png]]

[4]为什么将const用于引用返回类型：
![[Pasted image 20230728175556.png]]


##### （5）将引用用于类对象：
>将类对象传递给函数时，c++可以通过使用引用，让函数将类string、ostream......类的对象作为参数
```cpp
// strquote.cpp  -- different designs
#include <iostream>
#include <string>
using namespace std;
string version1(const string & s1, const string & s2);
const string & version2(string & s1, const string & s2);  // has side effect
const string & version3(string & s1, const string & s2);  // bad design

int main()
{
    string input;
    string copy;
    string result;

    cout << "Enter a string: ";
    getline(cin, input);
    copy = input;
    cout << "Your string as entered: " << input << endl;
    
    result = version1(input, "***");
    cout << "Your string enhanced: " << result << endl;
    cout << "Your original string: " << input << endl;

    result = version2(input, "###");
    cout << "Your string enhanced: " << result << endl;
    cout << "Your original string: " << input << endl;

    cout << "Resetting original string.\n";
    input = copy;
    result = version3(input, "@@@");
    cout << "Your string enhanced: " << result << endl;
    cout << "Your original string: " << input << endl;
    // cin.get();
    // cin.get();
    return 0;
}

string version1(const string & s1, const string & s2)
{
    string temp;    
    temp = s2 + s1 + s2;
    return temp;
}

const string & version2(string & s1, const string & s2)   // has side effect
{
    s1 = s2 + s1 + s2;
// safe to return reference passed to function
    return s1;
}

const string & version3(string & s1, const string & s2)   // bad design
{
    string temp;
    temp = s2 + s1 + s2;
// unsafe to return reference to local variable
    return temp;
}

结果：
Enter a string: dxgkuD
Your string as entered: dxgkuD
Your string enhanced: ***dxgkuD***
Your original string: dxgkuD
Your string enhanced: ###dxgkuD###
Your original string: ###dxgkuD###
Resetting original string.
```

程序分析：
version1:
![[Pasted image 20230728181450.png]]

version2:
![[Pasted image 20230728181655.png]]

version3:
![[Pasted image 20230728181810.png]]
>分析：
```cpp
const string& version3 (string& s1 , const string& s2)
(1) const 说明: version3函数返回值不被修改
(2) string& 说明返回值类型是string的引用
(3) 函数version3的模块内，先创建temp，再返回，函数要求的返回值是temp的引用。而temp的生命周期仅限在模块内。因此，执行顺序是：
	[1]创建temp，进行body操作
	[2]函数体执行结束，temp已经消亡，准备返回
	[3]返回值是string的引用，此时发现待返回的东西不见了！
	[4]报错！
```

##### （6）对象、继承、引用：
留给后面

##### （7）何时使用引用参数：
![[Pasted image 20230728182852.png]]


### 3）默认参数：

（1）定义：
>函数调用中省略了实参时，自动使用的一个值。
>eg: 如果写 void wow ( int n = 1 )，则函数调用 wow( ) 相当于wow(1)
>eg: char* left ( const char* str , int n = 1 )
>>[1] char*:    返回值是一个新的字符串
>>[2] 第一个参数使用const:    原始字符串保持不变
>>[3] 默认参数值是初始化值，因此将上面的原型将n设置为1  =>  如果省略参数n，它的值将为1；否则传递的值将覆盖1

（2）使用说明：
>[1] 对于带参数列表的函数，必须从右向左添加默认值。换言之，要为某个参数设置默认值，则必须为它右边的所有参数都设置默认值！
```cpp
int hbx1(int n , int m = 4 , int d = 5);      //valid
int hbx2(int n , int m = 4 , int d);          //invalid
int hbx3(int n = 1 , int m = 4 , int d = 9);  //valid

例如：调用 hbx1()函数 时，允许调用提供 1,2,3 个参数！
hhh = hbx1(2)      等价于：hbx(2,4,5)
hhh = hbx1(1,8)    等价于：hbx(1,8,5)
hhh = hbx1(8,7,6)  等价于：hbx(8,7,6)
```
>[2] 实参按照从左向右的顺序依次被赋给相应的形参，而不能跳过任何参数！
```cpp
这种调用就不被允许：
hhh = hbx1(3,,8)    //invalid , the computer cannnot put 4 to the blank!
```