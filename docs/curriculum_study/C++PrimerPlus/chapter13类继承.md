### 1）一个简单的基类

【1】基类原型：
```cpp
// tabtenn0.h -- a table-tennis base class
#ifndef TABTENN0_H_
#define TABTENN0_H_
#include <string>
using std::string;
// simple base class
class TableTennisPlayer
{
private:
    string firstname;
    string lastname;
    bool hasTable;
public:
    TableTennisPlayer (const string & fn = "none",
                       const string & ln = "none", bool ht = false);
    void Name() const;
    bool HasTable() const { return hasTable; };
    void ResetTable(bool v) { hasTable = v; };
};
#endif
```
----------------------------------------------------------------------------------------------------------------
```cpp
//tabtenn0.cpp -- simple base-class methods
#include "tabtenn0.h"
#include <iostream>
TableTennisPlayer::TableTennisPlayer (const string & fn,
    const string & ln, bool ht) : firstname(fn),
        lastname(ln), hasTable(ht) {}
void TableTennisPlayer::Name() const
{
    std::cout << lastname << ", " << firstname;
}
```
-----------
```cpp
// usett0.cpp -- using a base class
#include <iostream>
#include "tabtenn0.h"
int main ( void )
{
    using std::cout;
    TableTennisPlayer player1("Chuck", "Blizzard", true);
    TableTennisPlayer player2("Tara", "Boomdea", false);
    player1.Name();
    if (player1.HasTable())
        cout << ": has a table.\n";
    else
        cout << ": hasn't a table.\n";

    player2.Name();
    if (player2.HasTable())
        cout << ": has a table";
    else
        cout << ": hasn't a table.\n";
    // std::cin.get();
    return 0;
}
```
说明：
(1) 构造函数使用了初始化列表的方法，这在后续非常常见！
```cpp
原型中：
TableTennisPlayer (const string& fn="none", const string& ln="none", bool ht=false);

类外定义时：
TableTennisPlayer::TableTennisPlayer (const string& fn, const string& ln, bool ht) : firstname(fn),lastname(ln),hasTable(ht) {}
```
(2) 初始化列表的写法与常规构造函数写法等价！
```cpp
比如上述初始化列表等同于：
TableTennisPlayer::TableTennisPlayer (const string& fn, const string& ln, bool ht)
{
	firstname = fn;
	lastname = ln;
	hasTable = ht;
}
```
(3) string 与 char* 等价；


【2】派生一个类：
```cpp
// tabtenn1.h -- a table-tennis base class
#ifndef TABTENN1_H_
#define TABTENN1_H_
#include <string>
using std::string;
// simple base class
class TableTennisPlayer
{
private:
    string firstname;
    string lastname;
    bool hasTable;
public:
    TableTennisPlayer (const string & fn = "none",
                       const string & ln = "none", bool ht = false);
    void Name() const;
    bool HasTable() const { return hasTable; };
    void ResetTable(bool v) { hasTable = v; };
};
-----------------------------------------------------------------------------------------------------------------------------------
// simple derived class
class RatedPlayer : public TableTennisPlayer
{
private:
    unsigned int rating;
public:
    RatedPlayer (unsigned int r = 0, const string & fn = "none",
                 const string & ln = "none", bool ht = false);
    RatedPlayer(unsigned int r, const TableTennisPlayer & tp);
    unsigned int Rating() const { return rating; }
    void ResetRating (unsigned int r) {rating = r;}
};
#endif
```
---
```cpp
//tabtenn1.cpp -- simple base-class methods
#include "tabtenn1.h"
#include <iostream>
TableTennisPlayer::TableTennisPlayer (const string & fn,
    const string & ln, bool ht) : firstname(fn),
        lastname(ln), hasTable(ht) {}

void TableTennisPlayer::Name() const
{
    std::cout << lastname << ", " << firstname;
}

// RatedPlayer methods
RatedPlayer::RatedPlayer(unsigned int r, const string & fn,
     const string & ln, bool ht) : TableTennisPlayer(fn, ln, ht)
{
    rating = r;
}

RatedPlayer::RatedPlayer(unsigned int r, const TableTennisPlayer & tp)
    : TableTennisPlayer(tp), rating(r)
{
}
```
---
```cpp
// usett1.cpp -- using base class and derived class
#include <iostream>
#include "tabtenn1.h"
int main ( void )
{
    using std::cout;
    using std::endl;

    TableTennisPlayer player1("Tara", "Boomdea", false);
    RatedPlayer rplayer1(1140, "Mallory", "Duck", true);
   
    rplayer1.Name();          // derived object uses base method
    if (rplayer1.HasTable())
        cout << ": has a table.\n";
    else
        cout << ": hasn't a table.\n";
    
    player1.Name();           // base object uses base method
    if (player1.HasTable())
        cout << ": has a table";
    else
        cout << ": hasn't a table.\n";
    cout << "Name: ";

    rplayer1.Name();
    cout << "; Rating: " << rplayer1.Rating() << endl;
    
// initialize RatedPlayer using TableTennisPlayer object
    RatedPlayer rplayer2(1212, player1);
    cout << "Name: ";
    rplayer2.Name();
    cout << "; Rating: " << rplayer2.Rating() << endl;
    // std::cin.get();
    return 0;
}
```
---------
（1）派生类写法：
```cpp
class RatedPlayer : public TableTennisPlayer
{
	...
}

1) 指出RatedPlayer类的基类是TableTennis类
2) public说明是公有派生：基类的公有成员 将成为 派生类的公有成员；基类的私有成员也将成为派生类的一部分，但是只能通过基类的公有和保护方法访问！
```

（2）派生类需要在继承特性中添加说明：
>[1]  **派生类需要自己的构造函数**

>[2]  派生类可以根据需要添加额外的 数据成员 and 成员函数

（3）派生类构造函数的访问权限：
>[1] ==派生类不能直接访问基类的私有成员，必须通过基类的方法进行访问==

>[2] 派生类构造函数必须使用基类构造函数

>[3] 创建派生类对象时，程序必须首先创建基类对象（基类对象应当在程序进入派生类构造函数之前就被创建）
```cpp
RatedPlayer::RatedPlayer (unsigned int r, const string& fn, const string& ln, bool ht) : TableTennisPlayer(fn,ln,ht)
{
	rating = r;
}

:TableTennisPlayer(fn,ln,ht) 是成员初始化列表【基类构造函数】
该构造函数的执行顺序：
（1）RatedPlayer构造函数将把实参 "hbx","dusk","hhh" 赋给形参 fn,ln,ht
（2）将这些参数作为实参传递给TableTennisPlayer构造函数
（3）将创建一个TableTennisPlayer对象，并将数据 "hbx" , "dusk" , "hhh" 存储在该对象中
（4）程序进入构造函数体，完成RealPlayer对象的创建，并将参数r的值赋给rating成员
```

>[4] 如果忽略成员初始化列表，程序将使用基类的默认构造函数
```cpp
忽略初始化列表：
RatedPlayer::RatedPlayer(unsigned int r, const string& fn, const string& ln, bool ht)
{
	rating = r;
}
-------------------------------------------------------------------------------------------
等价于：
RatedPlayer::RatedPlayer(unsigned int r, const string& fn, const string& ln, bool ht) : TableTennisPlayer() 
{
	rating = r;
}
```

>[5] 初始化列表的参数改为类成员时
```cpp
RatedPlayer::RatedPlayer(unsigned int r, const TableTennisPlayer& tp) : TableTennisPlayer(tp)
{
	rating = r;
}

由于tp类型为TableTennisPlayer&，因此将调用基类的复制构造函数
基类没有定义复制构造函数，因此编译器将自动生成一个
这种情况下 隐式复制构造函数 是合适的
因为这个类没有使用 动态内存分配！
```

>[6] 为简化也可以对派生类成员使用 成员初始化列表 语法
>1）对于继承的基类，初始化列表用的是类名: TableTennisPlayer(tp)
>2）对于派生类待初始化的成员，初始化列表用的是成员名：rating(r)
```cpp
RatedPlayer::RatedPlayer(unsigned int r, const TableTennisPlayer& tp) : TableTennisPlayer(tp),rating(r) {}
```

【3】继承的基本总结：
1）派生类构造函数的要点：
>[1] 首先创建基类对象
>[2] 派生类构造函数通过 成员初始化列表 将基类信息传递给基类构造函数
>[3] 派生类构造函数应初始化派生类新增的数据成员

2）顺序：
>[1] 释放对象的顺序与创建对象的顺序相反！即：先执行派生类的析构函数，再自动调用基类的析构函数
>[2] ==创建派生类对象时，程序首先调用基类构造函数，然后再调用派生类构造函数（基类构造函数负责初始化继承的数据成员；派生类构造函数主要用于初始化新增的数据成员）==

3）派生类和基类的特殊关系：
>[1] 派生类对象可以使用基类的方法，条件是：方法不是私有的
>[2] 基类指针可以在不进行显式类型转换的情况下指向派生类成员。反之不然！
>[3] 基类引用可以在不进行显式类型转换的情况下引用派生类成员。反之不然！
>ps：==**基类指针/引用** 只能用于调用 **基类方法** ，不能调用**派生类方法**==


### 2）多态公有继承
（1）问题引入：
```cpp
// brass.h  -- bank account classes
#ifndef BRASS_H_
#define BRASS_H_
#include <string>
// Brass Account Class
class Brass
{
private:
    std::string fullName;
    long acctNum;
    double balance;
public:
    Brass(const std::string & s = "Nullbody", long an = -1,
                double bal = 0.0);
    void Deposit(double amt);
    virtual void Withdraw(double amt);
    double Balance() const;
    virtual void ViewAcct() const;
    virtual ~Brass() {}
};

//Brass Plus Account Class
class BrassPlus : public Brass
{
private:
    double maxLoan;
    double rate;
    double owesBank;
public:
    BrassPlus(const std::string & s = "Nullbody", long an = -1,
            double bal = 0.0, double ml = 500,
            double r = 0.11125);
    BrassPlus(const Brass & ba, double ml = 500,
                                double r = 0.11125);
    virtual void ViewAcct()const;
    virtual void Withdraw(double amt);
    void ResetMax(double m) { maxLoan = m; }
    void ResetRate(double r) { rate = r; };
    void ResetOwes() { owesBank = 0; }
};
#endif
```
---
分析：
【1】两个类都声明了 ViewAcct() 和 Withdraw() 方法，但是BrassPlus对象与Brass对象的方法行为是不同的！

【2】对于在两个类中行为相同的方法，如：Deposit() 和 Balance() ，则只在基类中声明！

【3】使用 **关键字virtual** 的原因：
>[1] 如果没有virtual，程序将根据 **引用类型 or 指针类型** 选择方法
>[2] 如果使用virtual，程序将根据 **引用对象 or 指针指向的对象 的类型** 来选择方法
```cpp
(1) 如果 ViewAcct() 不是虚的，则程序的行为如下：

Brass dom("Dominic Banker",11224,4183.45);
BrassPlus dot("Dorthy Banker",12118,2592.00);
Brass& b1 = dom;
Brass& b2 = dot;
b1.ViewAcct(); // use Brass::ViewAcct()
b2.ViewAcct(); // use Brass::ViewAcct()

//引用变量的类型为Brass，所以选择了Brass::ViewAcct()
---------------------------------------------------------------------------
(2) 如果 ViewAcct() 是虚的，则程序的行为如下：

Brass dom("Dominic Banker",11224,4183.45);
BrassPlus dot("Dorthy Banker",12118,2592.00);
Brass& b1 = dom;
Brass& b2 = dot;
b1.ViewAcct(); // use Brass::ViewAcct()
b2.ViewAcct(); // use BrassPlus::ViewAcct()

//两个引用的类型都是Brass，但是b2引用的是一个BrassPlus对象，所以它是BrassPlus::ViewAcct()
```

【4】经常在基类中将 *派生类会重新定义的* 方法声明为**虚方法**

【5】为基类声明一个虚析构函数是惯例，原因见后面


（2）主程序分析：
```cpp
// brass.cpp -- bank account class methods
#include <iostream>
#include "brass.h"
using std::cout;
using std::endl;
using std::string;

// formatting stuff
typedef std::ios_base::fmtflags format;
typedef std::streamsize precis;
format setFormat();
void restore(format f, precis p);

// Brass methods
Brass::Brass(const string & s, long an, double bal)
{
    fullName = s;
    acctNum = an;
    balance = bal;
}

void Brass::Deposit(double amt)
{
    if (amt < 0)
        cout << "Negative deposit not allowed; "
             << "deposit is cancelled.\n";
    else
        balance += amt;
}

void Brass::Withdraw(double amt)
{
    // set up ###.## format
    format initialState = setFormat();
    precis prec = cout.precision(2);
    if (amt < 0)
        cout << "Withdrawal amount must be positive; "
             << "withdrawal canceled.\n";
    else if (amt <= balance)
        balance -= amt;
    else
        cout << "Withdrawal amount of $" << amt
             << " exceeds your balance.\n"
             << "Withdrawal canceled.\n";

    restore(initialState, prec);
}

double Brass::Balance() const
{
    return balance;
}

void Brass::ViewAcct() const
{
     // set up ###.## format
    format initialState = setFormat();
    precis prec = cout.precision(2);
    cout << "Client: " << fullName << endl;
    cout << "Account Number: " << acctNum << endl;
    cout << "Balance: $" << balance << endl;
    restore(initialState, prec); // Restore original format
}

// BrassPlus Methods
BrassPlus::BrassPlus(const string & s, long an, double bal,
           double ml, double r) : Brass(s, an, bal)
{
    maxLoan = ml;
    owesBank = 0.0;
    rate = r;
}

BrassPlus::BrassPlus(const Brass & ba, double ml, double r)
           : Brass(ba)   // uses implicit copy constructor
{
    maxLoan = ml;
    owesBank = 0.0;
    rate = r;
}

// redefine how ViewAcct() works
void BrassPlus::ViewAcct() const
{
    // set up ###.## format
    format initialState = setFormat();
    precis prec = cout.precision(2);

    Brass::ViewAcct();   // display base portion
    cout << "Maximum loan: $" << maxLoan << endl;
    cout << "Owed to bank: $" << owesBank << endl;
    cout.precision(3);  // ###.### format
    cout << "Loan Rate: " << 100 * rate << "%\n";
    restore(initialState, prec);
}

// redefine how Withdraw() works
void BrassPlus::Withdraw(double amt)
{
    // set up ###.## format
    format initialState = setFormat();
    precis prec = cout.precision(2);

    double bal = Balance();
    if (amt <= bal)
        Brass::Withdraw(amt);
    else if ( amt <= bal + maxLoan - owesBank)
    {
        double advance = amt - bal;
        owesBank += advance * (1.0 + rate);
        cout << "Bank advance: $" << advance << endl;
        cout << "Finance charge: $" << advance * rate << endl;
        Deposit(advance);
        Brass::Withdraw(amt);
    }
    else
        cout << "Credit limit exceeded. Transaction cancelled.\n";
    restore(initialState, prec);
}

format setFormat()
{
    // set up ###.## format
    return cout.setf(std::ios_base::fixed,
                std::ios_base::floatfield);
}

void restore(format f, precis p)
{
    cout.setf(f, std::ios_base::floatfield);
    cout.precision(p);
}
```
---
```cpp
常规使用：
// usebrass1.cpp -- testing bank account classes
// compile with brass.cpp
#include <iostream>
#include "brass.h"

int main()
{
    using std::cout;
    using std::endl;

    Brass Piggy("Porcelot Pigg", 381299, 4000.00);
    BrassPlus Hoggy("Horatio Hogg", 382288, 3000.00);

    Piggy.ViewAcct();
    cout << endl;
    Hoggy.ViewAcct();
    cout << endl;
    cout << "Depositing $1000 into the Hogg Account:\n";
    Hoggy.Deposit(1000.00);
    cout << "New balance: $" << Hoggy.Balance() << endl;
    cout << "Withdrawing $4200 from the Pigg Account:\n";
    Piggy.Withdraw(4200.00);
    cout << "Pigg account balance: $" << Piggy.Balance() << endl;
    cout << "Withdrawing $4200 from the Hogg Account:\n";
    Hoggy.Withdraw(4200.00);
    Hoggy.ViewAcct();
    // std::cin.get();
    return 0;
}
```
---
```cpp
演示虚方法：
// usebrass2.cpp -- polymorphic example
// compile with brass.cpp
#include <iostream>
#include <string>
#include "brass.h"
const int CLIENTS = 4;
int main()
{
   using std::cin;
   using std::cout;
   using std::endl;

   Brass * p_clients[CLIENTS];
   std::string temp;
   long tempnum;
   double tempbal;
   char kind;

   for (int i = 0; i < CLIENTS; i++)
   {
       cout << "Enter client's name: ";
       getline(cin,temp);

       cout << "Enter client's account number: ";
       cin >> tempnum;

       cout << "Enter opening balance: $";
       cin >> tempbal;

       cout << "Enter 1 for Brass Account or "
            << "2 for BrassPlus Account: ";

       while (cin >> kind && (kind != '1' && kind != '2'))
           cout <<"Enter either 1 or 2: ";
       if (kind == '1')
           p_clients[i] = new Brass(temp, tempnum, tempbal);
       else
       {
           double tmax, trate;
           cout << "Enter the overdraft limit: $";

           cin >> tmax;
           cout << "Enter the interest rate "
                << "as a decimal fraction: ";
           cin >> trate;

           p_clients[i] = new BrassPlus(temp, tempnum, tempbal,
                                        tmax, trate);
        }

        while (cin.get() != '\n')
            continue;
   }
   cout << endl;
   
   for (int i = 0; i < CLIENTS; i++)
   {
       p_clients[i]->ViewAcct();
       cout << endl;
   }
   
   for (int i = 0; i < CLIENTS; i++)
   {
       delete p_clients[i];  // free memory
   }
   cout << "Done.\n";        
 /* code to keep window open
   if (!cin)
      cin.clear();
   while (cin.get() != '\n')
      continue;
*/
   return 0;
}
```
---

分析：
1）**关键字virtual**只用于类声明的方法原型中，而没有用于方法定义中
```cpp
看这两个虚函数的方法定义：
void Brass::Withdraw(double amt) {...}
void Brass::ViewAcct() const {...}
void BrassPlus::ViewAcct() const {...}
void BrassPlus::withdraw(double amt) {...}
```

2）派生类不能**直接访问**基类的私有数据，必须使用**基类的公有方法**，访问的方式取决于方法
>[1] 派生类构造函数在初始化 ==基类私有数据== 时，采用的是==成员初始化列表==的语法
```cpp
// BrassPlus Methods
形式(1)
BrassPlus::BrassPlus(const string & s, long an, double bal,
           double ml, double r) : Brass(s, an, bal)
{
    maxLoan = ml;
    owesBank = 0.0;
    rate = r;
}
-----------------------------------------------------------------------
形式(2)
BrassPlus::BrassPlus(const Brass & ba, double ml, double r)
           : Brass(ba)   // uses implicit copy constructor
{
    maxLoan = ml;
    owesBank = 0.0;
    rate = r;
}

在这里：const Brass& ba 就是对上文 const string & s + long an + double bal的类对象概括
```

>[2] 非构造函数不能使用初始化列表，但是派生类方法**可以调用公有的基类方法**
```cpp
// redefine how ViewAcct() works
void BrassPlus::ViewAcct() const
{
    // set up ###.## format
    format initialState = setFormat();
    precis prec = cout.precision(2);

    Brass::ViewAcct();   // display base portion             // 演示 基类成员信息
    cout << "Maximum loan: $" << maxLoan << endl;            // 演示 派生类新增成员信息
    cout << "Owed to bank: $" << owesBank << endl;           // 演示 派生类新增成员信息
    cout.precision(3);  // ###.### format
    cout << "Loan Rate: " << 100 * rate << "%\n";            // 演示 派生类新增成员信息
    restore(initialState, prec);
}

// redefine how Withdraw() works
void BrassPlus::Withdraw(double amt)
{
    // set up ###.## format
    format initialState = setFormat();
    precis prec = cout.precision(2);

    double bal = Balance();                                         // 演示 基类成员信息
    if (amt <= bal)
        Brass::Withdraw(amt);                                       // 演示 基类成员信息
    else if ( amt <= bal + maxLoan - owesBank)
    {
        double advance = amt - bal;
        owesBank += advance * (1.0 + rate);
        cout << "Bank advance: $" << advance << endl;
        cout << "Finance charge: $" << advance * rate << endl;
        Deposit(advance);
        Brass::Withdraw(amt);                                       // 演示 基类成员信息
    }
    else
        cout << "Credit limit exceeded. Transaction cancelled.\n";
    restore(initialState, prec);
}

--------------------------------------------------------------------------------------------
解释：关于作用域解析运算符::的问题

（1） Brass::ViewAcct();   // display base portion
说明： BrassPlus::ViewAcct() 显示新增的BrassPlus数据成员，并调用基类方法 Brass::ViewAcct() 来显示基类数据成员
这里必须加上 :: ，因为 ViewAcct() 函数存在二义性！

（2） double bal = Balance();   // 演示 基类成员信息
说明1： 使用基类的 Balance() 函数确定余额，因为派生类没有重新定义该方法，代码不必对 balance() 使用 :: 【不存在二义性！】
说明2： 这里能够自动识别出来是 Brass::Balance() 还因为：派生类能调用基类的公有方法！
```

>[3] 多态性演示：
![[Pasted image 20230803150341.png]]
```cpp
演示虚方法：
// usebrass2.cpp -- polymorphic example
// compile with brass.cpp
#include <iostream>
#include <string>
#include "brass.h"
const int CLIENTS = 4;
int main()
{
   using std::cin;
   using std::cout;
   using std::endl;

   Brass * p_clients[CLIENTS];----------------------------------- // 建立 指向Brass类型的指针 数组
   std::string temp;
   long tempnum;
   double tempbal;
   char kind;

   for (int i = 0; i < CLIENTS; i++)
   {
       cout << "Enter client's name: ";
       getline(cin,temp);

       cout << "Enter client's account number: ";
       cin >> tempnum;

       cout << "Enter opening balance: $";
       cin >> tempbal;

       cout << "Enter 1 for Brass Account or "
            << "2 for BrassPlus Account: ";

       while (cin >> kind && (kind != '1' && kind != '2'))
           cout <<"Enter either 1 or 2: ";
       if (kind == '1')
           p_clients[i] = new Brass(temp, tempnum, tempbal);------// 新建 Brass对象
       else
       {
           double tmax, trate;
           cout << "Enter the overdraft limit: $";

           cin >> tmax;
           cout << "Enter the interest rate "
                << "as a decimal fraction: ";
           cin >> trate;

           p_clients[i] = new BrassPlus(temp, tempnum, tempbal,---// 新建 BrassPlus对象
                                        tmax, trate);
        }

        while (cin.get() != '\n')
            continue;
   }
   cout << endl;
   
   for (int i = 0; i < CLIENTS; i++)
   {
       p_clients[i]->ViewAcct();---------------------------------// 输出 数组对象（Brass + BrassPlus）
       cout << endl;
   }
   
   for (int i = 0; i < CLIENTS; i++)
   {
       delete p_clients[i];     free memory----------------------// 删除 数组对象（Brass + BrassPlus）
   }
   cout << "Done.\n";        
 /* code to keep window open
   if (!cin)
      cin.clear();
   while (cin.get() != '\n')
      continue;
*/
   return 0;
}
```
多态性的体现在于：
```cpp
   for (int i = 0; i < CLIENTS; i++)
   {
       p_clients[i]->ViewAcct();---------------------------------// 输出 数组对象（Brass + BrassPlus）
       cout << endl;
   }
```
**如果数组成员指向的是Brass对象，则调动 Brass : : ViewAcct( ) ；如果指向的是BrassPlus对象，则调用 BrassPlus : : ViewAcct( )**
ps: 如果 BrassPlus : : Acct( )没有被声明为虚的，则在任何情况下都是按照指针类型处理，即调用的是 Brass : : ViewAcct( )

>[4] 为什么需要设置虚析构函数：
如果指针指向的是BrassPlus对象，将调用BrassPlus的析构函数，然后自动调用基类的析构函数
【使用虚析构函数可以确保正确的析构函数序列被调用，必须设置！】

### 3）静态联编和动态联编
##### （1）引入：
函数名联编：将源代码中的函数调用解释为执行特定的函数代码块
静态联编：在编译过程中进行的联编（static binding）
动态联编：使用哪一个函数是不能在编译时确定的，因为编译器不知道用户将选择哪种类型的对象。编译器必须生成能够在程序运行时选择正确的虚函数的代码（dynamic binding）

##### （2）指针和引用类型的兼容性：
【1】通常 c++ 不允许将一种类型的地址赋给另一种类型的指针，也不允许一种类型的引用指向另一种类型
```cpp
double x = 2.5;
int* pi = &x;  //invalid!  double -> int
long& rl = x;  //invalid!  double -> long
```

【2】指向**基类的引用or指针**可以引用**派生类对象**，而不必进行显示类型转换——向上强制转换(upcasting)
```cpp
BrassPlus hbx("shhss",498888,549);
Brass* pb = &hbx;  //ok
Brass& rb = hbx;   //ok

将 派生类引用or指针 转换为 基类引用or指针 被称为：向上强制转换(upcasting)，这使得公有继承不需要进行显式类型转换
```
![[Pasted image 20230804102112.png]]

【3】将 **基类指针or引用** 转换为 **派生类指针or引用** ——向下强制转换(downcasting)，必须显式类型转换！

【4】对于使用基类引用or指针作为参数的函数调用，将进行upcasting
![[Pasted image 20230804102524.png]]
说明：按值传递只将BrassPlus对象的Brass部分传递给函数 fv( ) 。但随引用和指针发生的upcasting导致fr( )与fp( ) 分别为Brass对象和BrassPlus对象使用Brass : : ViewAcct() 和 BrassPlus : : ViewAcct()

##### （3）虚函数工作原理：
![[Pasted image 20230804103849.png]]
![[Pasted image 20230804103905.png]]
![[Pasted image 20230804104020.png]]

##### （4）虚函数的注意事项：
【基础知识】
![[Pasted image 20230804104253.png]]

-----
【深入了解】
（1）构造函数 
>**构造函数不能是虚函数**
>创建派生类对象时，将调用派生类的构造函数，而不是基类的构造函数，然后，派生类的构造函数将使用基类的一个构造函数，这种顺序不是继承机制

（2）析构函数
>**析构函数必须是虚函数**（通常应给基类加一个虚析构函数，即使它并不需要析构）
![[Pasted image 20230804104802.png]]

（3）友元
>**友元不能是虚函数，因为友元不是类成员，只有成员才能是虚函数**

（4）重新定义将隐藏方法
![[Pasted image 20230804105132.png]]
![[Pasted image 20230804105320.png]]
![[Pasted image 20230804105355.png]]
![[Pasted image 20230804105408.png]]


### 4）访问控制：protected
**（1）关键字protected与private相似，在类外只有用公有类成员来访问protected部分的类成员
（2）private与protected之间的区别：派生类成员可以直接访问基类的protected成员；但不能直接访问基类的私有成员**
![[Pasted image 20230804105727.png]]
![[Pasted image 20230804105815.png]]


### 5）抽象基类（abstract base class，ABC）
###### （1）引入：
![[Pasted image 20230804111915.png]]
![[Pasted image 20230804112147.png]]
![[Pasted image 20230804112158.png]]

---
![[Pasted image 20230804112243.png]]

###### （2）示例应用：
```cpp
// acctabc.h  -- bank account classes
#ifndef ACCTABC_H_
#define ACCTABC_H_
#include <iostream>
#include <string>

// Abstract Base Class
class AcctABC
{
private:
    std::string fullName;
    long acctNum;
    double balance;

protected:
    struct Formatting
    {
         std::ios_base::fmtflags flag;
         std::streamsize pr;
    };
    const std::string & FullName() const {return fullName;}
    long AcctNum() const {return acctNum;}
    Formatting SetFormat() const;
    void Restore(Formatting & f) const;

public:
    AcctABC(const std::string & s = "Nullbody", long an = -1,
                double bal = 0.0);
    void Deposit(double amt) ;
    virtual void Withdraw(double amt) = 0; -------------------------------------- pure virtual function
    double Balance() const {return balance;};
    virtual void ViewAcct() const = 0;     -------------------------------------- pure virtual function
    virtual ~AcctABC() {}                  -------------------------------------- 别忘了虚析构函数
};

// Brass Account Class
class Brass :public AcctABC
{
public:
    Brass(const std::string & s = "Nullbody", long an = -1,
           double bal = 0.0) : AcctABC(s, an, bal) { }     ------------------------普通的就直接继承ABC
    virtual void Withdraw(double amt);                     ------------------------继承ABC的纯虚函数 => 虚函数
    virtual void ViewAcct() const;                         ------------------------继承ABC的纯虚函数 => 虚函数
    virtual ~Brass() {}
};

//Brass Plus Account Class
class BrassPlus : public AcctABC
{
private:
    double maxLoan;
    double rate;
    double owesBank;                      ---------------------------------------- 新增 3 个数据成员

public:
    BrassPlus(const std::string & s = "Nullbody", long an = -1,
            double bal = 0.0, double ml = 500,
            double r = 0.10);                              ------------------------普通的就直接继承ABC
    BrassPlus(const Brass & ba, double ml = 500, double r = 0.1);
    virtual void ViewAcct()const;                          ------------------------继承ABC的纯虚函数 => 虚函数
    virtual void Withdraw(double amt);                     ------------------------继承ABC的纯虚函数 => 虚函数
    void ResetMax(double m) { maxLoan = m; }
    void ResetRate(double r) { rate = r; };
    void ResetOwes() { owesBank = 0; }                     ------------------------新增 3 个派生类成员函数
};
#endif
```
---
```cpp
// acctabc.cpp -- bank account class methods
#include <iostream>
#include "acctabc.h"
using std::cout;
using std::ios_base;
using std::endl;
using std::string;

// Abstract Base Class
AcctABC::AcctABC(const string & s, long an, double bal)--------------------------ABC抽象基类的初始化函数
{
    fullName = s;
    acctNum = an;
    balance = bal;
}

void AcctABC::Deposit(double amt)------------------------------------------------ABC抽象基类的某成员函数
{
    if (amt < 0)
        cout << "Negative deposit not allowed; "
             << "deposit is cancelled.\n";
    else
        balance += amt;
}

void AcctABC::Withdraw(double amt)
{
    balance -= amt;
}

// protected methods for formatting
AcctABC::Formatting AcctABC::SetFormat() const
{
 // set up ###.## format
    Formatting f;
    f.flag =
        cout.setf(ios_base::fixed, ios_base::floatfield);
    f.pr = cout.precision(2);
    return f;
}

void AcctABC::Restore(Formatting & f) const
{
    cout.setf(f.flag, ios_base::floatfield);
    cout.precision(f.pr);
}

// Brass methods
void Brass::Withdraw(double amt)---------------------------------------------派生类1的某成员函数（虚函数）
{
    if (amt < 0)
        cout << "Withdrawal amount must be positive; "
             << "withdrawal canceled.\n";
    else if (amt <= Balance())-----------------------------------------------继承的是ABC基类，因此可以直接调用基类方法Balance()
        AcctABC::Withdraw(amt);----------------------------------------------继承的是ABC基类，因此可以调用基类方法Withdraw()
    else
        cout << "Withdrawal amount of $" << amt
             << " exceeds your balance.\n"
             << "Withdrawal canceled.\n";
}

void Brass::ViewAcct() const    ---------------------------------------------派生类1的某成员函数（虚函数）
{
    Formatting f = SetFormat();
    cout << "Brass Client: " << FullName() << endl;     ---------------------继承的是ABC基类，因此可以直接调用基类方法FullName()
    cout << "Account Number: " << AcctNum() << endl;    ---------------------继承的是ABC基类，因此可以直接调用基类方法AcctNum()
    cout << "Balance: $" << Balance() << endl;          ---------------------继承的是ABC基类，因此可以直接调用基类方法Balance()
    Restore(f);
}

// BrassPlus Methods
BrassPlus::BrassPlus(const string & s, long an, double bal,
           double ml, double r) : AcctABC(s, an, bal)
{
    maxLoan = ml;
    owesBank = 0.0;
    rate = r;
}

BrassPlus::BrassPlus(const Brass & ba, double ml, double r)
           : AcctABC(ba)   // uses implicit copy constructor
{
    maxLoan = ml;
    owesBank = 0.0;
    rate = r;
}

void BrassPlus::ViewAcct() const
{
    Formatting f = SetFormat();

    cout << "BrassPlus Client: " << FullName() << endl;
    cout << "Account Number: " << AcctNum() << endl;
    cout << "Balance: $" << Balance() << endl;
    cout << "Maximum loan: $" << maxLoan << endl;
    cout << "Owed to bank: $" << owesBank << endl;
    cout.precision(3);
    cout << "Loan Rate: " << 100 * rate << "%\n";
    Restore(f);
}

void BrassPlus::Withdraw(double amt)
{
    Formatting f = SetFormat();

    double bal = Balance();
    if (amt <= bal)
        AcctABC::Withdraw(amt);
    else if ( amt <= bal + maxLoan - owesBank)
    {
        double advance = amt - bal;
        owesBank += advance * (1.0 + rate);
        cout << "Bank advance: $" << advance << endl;
        cout << "Finance charge: $" << advance * rate << endl;
        Deposit(advance);
        AcctABC::Withdraw(amt);
    }
    else
        cout << "Credit limit exceeded. Transaction cancelled.\n";
    Restore(f); 
}
```
---
**总结**：
（0）ABC基类中的纯虚函数声明格式：
```cpp
virtual Func_Name(arguments1 , ...) = 0;    // = 0 ：a pure virtual function
```
（1）ABC作为基类的目的是：
>将共有元素、方法集中包装在一起，方便后续基于此进行个性化定制（抽象类 -> 具体类）

（2）派生类继承ABC基类时：
>【1】非“纯虚函数”继承：类中声明不需要virtual，定义格式同正常继承

>【2】纯虚函数”继承：类中声明需要virtual，定义格式也同正常继承
```cpp
void Brass::ViewAcct() const    ---------------------------------------------派生类1的某成员函数（虚函数）
{
    Formatting f = SetFormat();
    cout << "Brass Client: " << FullName() << endl;     ---------------------继承的是ABC基类，因此可以直接调用基类方法FullName()
    cout << "Account Number: " << AcctNum() << endl;    ---------------------继承的是ABC基类，因此可以直接调用基类方法AcctNum()
    cout << "Balance: $" << Balance() << endl;          ---------------------继承的是ABC基类，因此可以直接调用基类方法Balance()
    Restore(f);
}
```

（3）进行 *ABC——具体类（1,2,3...）* 的步骤与原则【==参见上面的实例！==】：
>[1]设计ABC基类
>>【1】纯虚函数记得写好声明virtual ... = 0；类外定义时不需要virtual
>>【2】其余函数正常声明与定义，与正常基类没区别

>[2]设计具体类(ABC---concrete)
>>【1】纯虚函数”继承：类中声明需要virtual，定义格式也同正常继承
>>【2】非“纯虚函数”继承：类中声明不需要virtual，定义格式同正常继承
>>【3】自定义新函数方法：与正常基类没区别

（4）ABC理念:
![[Pasted image 20230804120546.png]]


### 6）继承与动态内存分配
![[Pasted image 20230804140015.png]]
##### （1）派生类不使用new
>从baseDMA类中派生出lacksDMA类，如果后者不使用new，也没特殊设计，就不需要为lackDMA类设计：显式析构函数+复制构造函数+赋值运算符

##### （2）派生类使用new
>从baseDMA类中派生出lacksDMA类，如果后者使用new，或者有特殊设计，需要为lackDMA类设计：显式析构函数+复制构造函数+赋值运算符

假如派生类使用了new：
```cpp
class hasDMA : public baseDMA
{
private:
	char* style;
public:
	......
}
```
---
【1】析构函数
```cpp
baseDMA :: ~baseDMA()  // take care of baseDMA stuff
{
	delete [] label;
}


hasDMA :: ~hasDMA()    // take care of hasDMA stuff
{
	delete [] style;
}
```
---
【2】复制构造函数
```cpp
baseDMA :: baseDMA (const baseDMA& rs)                ---------【基类的复制构造函数】
{
	label = new char[std::strlen(rs.label) + 1];      ---------指针的拷贝
	std::strcpy(label,rs.label);
	
	rating = rs.rating;                  ----------------------非指针类型的拷贝
}

----------------------------------------------------------------------------------------------------------------------------------

hasDMA :: hasDMA (const hasDMA& hs) : baseDMA(hs)     ---------【派生类的复制构造函数】
{
    // baseDMA部分：已经由上面的:baseDMA(hs)说明了拷贝！
    //非baseDMA部分：
	style = new char[std::strlen(hs.style) + 1];
	std::strcpy(style,hs.style);                       --------指针的拷贝
}
// 这里（派生类）需要注意：
// 初始化列表将一个 hasDMA引用 传递给 baseDMA构造函数 
// 虽然没有参数类型为 hasDMA引用 的 baseDMA构造函数，但是基类引用可以指向派生类型
// 因此：baseDMA复制构造函数将使用 hasDMA参数 的 baseDMA部分 来构造baseDMA部分！
```
---
【3】赋值运算符
```cpp
基类的赋值运算符：
baseDMA& baseDMA :: operator=(const baseDMA& rs)
{
	if(this == &rs) return *this;                   // 地址相同：直接返回即可
	
	delete [] label;                                // 先清理这块内存

    // 指针类型拷贝：
    label = new char[std::strlen(rs.label) + 1];    // 开辟新空间
    std::strcpy(label,rs.label);                    // str拷贝进新内存处

    // 非指针类型拷贝：
	rating = rs.rating;
	
	return *this;                                   // 返回对象
}

----------------------------------------------------------------------------------------------------------------------------------
派生类的赋值运算符：
hasDMA& hasDMA :: operator=(const hasDMA& hs)       // 引用对象hs是hasDMA类型(baseDMA + 其他)
{
	if(this == &hs) return *this;                   // 地址相同：直接返回即可
    baseDMA::operator=(hs);                         // 将 hs中包含的baseDMA成员 拷贝
    
    delete [] style;                                // 先清理这块内存
    
    style = new char [std::strlen(hs.style) + 1];   // 开辟新空间
    std::strcpy(style,hs.style);                    // str拷贝进新内存处
    return *this;
}
```

### 7）类设计回顾

略！