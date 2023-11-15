### 1）数组的复习：

>（1）只有定义数组的时候才可以初始化，此后就不可以使用了； 
		 不能将一个array赋给另一个array！
		 
>（2）初始化数组，提供值可以少于数组元素个数
		这种针对数组部分初始化的行为，编译器自动将其他元素补成 0 【不在乎数组定义在main()的内/外】
```cpp
1) 写法一：[在函数体外面]
# include<iostream>
using namespace std;
int a[10]={1,1};
int main() {
    for(int i=0;i<=9;i++) cout << a[i] <<' ';
    return 0;
}

2) 写法二：[在函数体内部]
# include<iostream>
using namespace std;
int main() {
    int a[10]={1,1};

    for(int i=0;i<=9;i++) cout << a[i] <<' ';
    return 0;
}

结果都是：
1 1 0 0 0 0 0 0 0 0
```

>（3）数组元素全部赋值成0： int  a[100] = { 0 }; 
		reason: 显式地将第一个元素设置为0，其余元素编译器按(2)中规则自动赋值为0
		ps：该写法等价于 int a[100] = { };    [ 括号内置为空 ]
		tips：int  a[100] = {1}; 含义是 第一个元素 a[0] 是1，其余元素都是0
		
>（4）初始化时[ ]写法，编译器将自动计算元素个数：
         int ar[ ] = {1,2,3,99};          // 自动计算出num=4
```cpp
#include<iostream> 
using namespace std;
int main() 
{
    int a[] = {1,4,3,6,5,8,0};
    int num = sizeof(a)/sizeof(int);
    
	cout << num <<endl;
	return 0;
}     
结果：  7
```

### 2) 字符串（char）：

>（1）char 的字符数组与字符串：
		字符数组：char dog[8] = {'a','b','c','d','e','f','g','r'};       // not a string
		字符串：    char cat[8] = {'a','b','c','d','e','f','g', '\0' };   // is a string

>>[1] 打印cat时,打印到'\0'空字符就结束，因此只显示abcdefg
>>[2] 打印dog时,打印完字符数组元素后，接着打印出内存中的‘随后字符’，直到遇见'\0'为止
>>==由于空字符'\0'在内存中非常常见，因此此过程很快就停止！==
```cpp
#include<iostream>
using namespace std;
int main() 
{
    char cat[8] = {'a','b','c','d','e','f','g', '\0' };    // is a string
    cout << cat <<'\n';

    char dog[8] = {'a','b','c','d','e','f','g','r'};       // not a string
    cout << dog <<'\n';
    return 0;
}

结果：
abcdefg
abcdefgrabcdefg
```

>（2）对于字符串的写法改进：
```cpp
char bio[11] = "mr. cheeps";     // 10个元素 + 1个'\0'
char fish[] = "dujkadfukasgbdf"  // 编译器自动计算元素个数
```
ps：    “ ” 与 ’ ‘不能互换！

>（3）字符串拼接时，不会在被连接的str之间自动添加空格，后者紧跟前者

>（4）字符串部分初始化后，其余元素自动赋值为'\0' :
```cpp
#include<iostream>
using namespace std;
int main()
{
    char cat[8] = {'a','b','c','d'};   // is a string

    cout << cat <<'\n';
    return 0;
}

结果：   abcd
```

>（5）strlen() 与 sizeof() 区别：
>>sizeof( )  :  整个数组的长度                          [ 取决于开设状态 ]
>>strlen( )  :  存储在数组中的字符串的长度    [ 取决于有效元素个数 ]
```cpp
// strings.cpp -- storing strings in an array
#include <iostream>
#include <cstring>  // for the strlen() function
int main()
{
    using namespace std;
    const int Size = 15;

    char name1[Size];               // empty array
    char name2[Size] = "C++owboy";  // initialized array
    // NOTE: some implementations may require the static keyword
    // to initialize the array name2
    cout << "Howdy! I'm " << name2;
    cout << "! What's your name?\n";
    
    cin >> name1;
    cout << "Well, " << name1 << ", your name has ";
    cout << strlen(name1) << " letters and is stored\n";
    cout << "in an array of " << sizeof(name1) << " bytes.\n";
    cout << "Your initial is " << name1[0] << ".\n";

    name2[3] = '\0';                // set to null character
    cout << "Here are the first 3 characters of my name: ";
    cout << name2 << endl;
    // cin.get();
    // cin.get();
    return 0;
}

结果：
Howdy! I'm C++owboy! What's your name?
hbx
Well, hbx, your name has 3 letters and is stored
in an array of 15 bytes.
Your initial is h.
Here are the first 3 characters of my name: C++
```

（6）字符串输入：
>【1】引入：
```cpp
// instr1.cpp -- reading more than one string
#include <iostream>
int main()
{
    using namespace std;
    const int ArSize = 20;
    char name[ArSize];
    char dessert[ArSize];

    cout << "Enter your name:\n";
    cin >> name;

    cout << "Enter your favorite dessert:\n";
    cin >> dessert;

    cout << "I have some delicious " << dessert;
    cout << " for you, " << name << ".\n";
    // cin.get();
    // cin.get();
    return 0;
}

结果：
Enter your name:
Alistair Dreeb
Enter your favorite dessert:
I have some delicious Dreeb for you, Alistair.
```
>发现问题：
我们都没有机会输入dessert，程序便将它显示出来，并且终止程序了，显而易见：
想输出的dessert变成了这个人的姓氏，实际上输出的人名也是他的first name

>分析问题：
>>1）cin使用 空白(空格,制表符,换行符)作为 str 结束的终止位置
>>2）这就意味着：cin只能获取第一个单词
>>3）本例中：cin 将Alistair作为第一个字符串，并将其放置于name数组中；因此Dreeb留在输入队列中；当cin在输入队列中搜索dessert时，首先看到的是Dreeb；因此cin读取的是Dreeb，并将其放置于dessert数组中

解决问题：
对于 New York 、Harvard University等短语，我们需要的是面向行而不是面向单词的方法
=>   使用getline() 与 get()
这两者都读取一行输入，直到到达换行符'\n'，随后getline()将换行符替换成'\0'【等价于丢弃换行符】，而get()将换行符保留在输入序列中！

>【2】getline()：
![[Pasted image 20230726161716.png]]
格式：cin.getline(arr_name , max_num)
机理：getline每次读取一行，它通过换行符确定行尾，但是不保存它；在存储字符串时，它将空字符’  \\0 ‘代替换行符！

>>>>>>>>>>>>![[Pasted image 20230726162058.png]]

>【3】get():
![[Pasted image 20230726163507.png]]
格式：cin.get(arr_name , max_size)
==ps【解释上述code】==
>>(1)    cin.get(name,Arsize);        //  输入name，末尾带上一个'\\n'
>>(2)    cin.get();                           //  吞掉前面遗留的'\\n'(自己本身不含有'\\n')
>>(3)    cin.get(dessert,Arsize);     //  输入dessert，末尾带上一个'\\n'

>【4】混合输入数字与str导致的问题：
```cpp
// numstr.cpp -- following number input with line input
#include <iostream>
int main()
{
    using namespace std;
    
    cout << "What year was your house built?\n";
    int year;
    cin >> year;
    // cin.get();
    cout << "What is its street address?\n";
    
    char address[80];
    cin.getline(address, 80);

    cout << "Year built: " << year << endl;
    cout << "Address: " << address << endl;
    cout << "Done!\n";
    // cin.get();
    return 0;
}

结果：
What year was your house built?
2004
What is its street address?
Year built: 2004
Address: 
Done!
```
problem：我们根本没有输入addresss的机会
analysis：cin读取年份后，将回车键生成的换行符'\n'留在了输入队列中，后面cin.getline(...)看见换行符，认为这是一个空行，并将一个空串赋给address数组
solve：在读取add之前先丢弃换行符'\\n'
```cpp
cin >> year;
cin.get()
...

结果：
What year was your house built?
2004
What is its street address?
hf
Year built: 2004
Address: hf
Done!
```

### 3) 字符串（string）：
（1）初始化，赋值，拼接，附加...      过于简单，略之！
（2）输入/输出：
面向单词：cin、cout
面向行：getline(cin , str)
```cpp
// strtype4.cpp -- line input
#include <iostream>
#include <string>               // make string class available
#include <cstring>              // C-style string library

int main()
{
    using namespace std;
    char charr[20];
    string str;

    cout << "Length of string in charr before input: "
         << strlen(charr) << endl;
    cout << "Length of string in str before input: "
         << str.size() << endl;
    cout << "Enter a line of text:\n";
    cin.getline(charr, 20);     // indicate maximum length
    cout << "You entered: " << charr << endl;
    
    cout << "Enter another line of text:\n";
    getline(cin, str);          // cin now an argument; no length specifier

    cout << "You entered: " << str << endl;
    cout << "Length of string in charr after input: "
         << strlen(charr) << endl;
    cout << "Length of string in str after input: "
         << str.size() << endl;
    return 0;
}

结果：
Length of string in charr before input: 1
Length of string in str before input: 0
说明char的strlen长度是考虑'\0'的，而string不考虑！

Enter a line of text:
hsauoigcxoasgdxo
You entered: hsauoigcxoasgdxo

Enter another line of text:
asdjh jdsilhids
You entered: asdjh jdsilhids
Length of string in charr after input: 16
Length of string in str after input: 15
```

### 4) 结构体：
基础部分省略，这里着重介绍程序分析：
```cpp
// structur.cpp -- a simple structure
#include <iostream>
using namespace std;
struct inflatable {   // structure declaration
    char name[20];
    float volume;
    double price;
};

int main()
{
    inflatable guest =
    {
        "Glorious Gloria",  // name value
        1.88,               // volume value
        29.99               // price value
    };  // guest is a structure variable of type inflatable
// It's initialized to the indicated values

    inflatable pal =
    {
        "Audacious Arthur",
        3.12,
        32.99
    };  // pal is a second variable of type inflatable
// NOTE: some implementations require using
// static inflatable guest =
    cout << "Expand your guest list with " << guest.name;
    cout << " and " << pal.name << "!\n";
// pal.name is the name member of the pal variable\
    cout << "You can have both for $";
    cout << guest.price + pal.price << "!\n";
    // cin.get();
    return 0;
}
```
分析：
（1）关键字struct表明，这些代码表示的是一个结构的布局；
（2）标识符inflatable是这种数据格式的名称，因此新类型的名称就叫做inflatable，它跟int/char......地位相当；
（3）结构声明定义了一种新类型，可以使用成员运算符( . )访问各个成员
（4）注意初始化方法：
```cpp
inflatable guest =
{
    "Glorious Gloria",  // name value
    1.88,               // volume value
    29.99               // price value
};  // guest is a structure variable of type inflatable

写法二：
inflatable guest = {"Glorious Gloria",1.88,29.99};

----------------------------------------------------------------------------------
ps: 如果大括号内为空，则各个成员设置为0【数字赋0 / 字符的字节设置为0】
inflatable hbx = {};
```

### 5) 共用体：
共用体（union）能够存储不同的数据类型，但是只能同时存储其中的一种类型
exp：（1）struct能同时存储int、char、double ; （2）union存储int/char/double
eg:
```cpp
union one4all                          // 声明它可以替代哪些
{
    int int_val;    
    long long_val;
    double double_val;
};

one4all pail;
pail.int_val = 15;                    // store an int 
cout << pail.int_val <<endl;

pail.double_val = 1.38;               //store a double && int value is lost!!!
cout << pail.double_val <<endl;
```

### 6) 枚举：
enum工具：创建符号常量，可以代替const
评价：限制严格，范围狭小，不太好用！

设置枚举值的规则：
（1）指定值必须是整数：int / long / long long
```cpp
enum hbx {one=1, two=2, three=3};
```
（2）默认情况下，首元素设置为 0 
（3）默认情况下，后面没有被初始化的枚举量的值比它的前一元素大1
```cpp
enum hbx_niu {first,second=100,third};
由(2)与(3),这里的first=0，third=101
```
（4）可以创建多个值相同的枚举量【means：它的实现机理不是集合】

### 7) 指针和自由存储空间：
（1）指针是一个变量，其存储的是值的地址，而不是值本身
（2）这种存储数据的策略是：将地址视为指定量，而将值视为派生量
（3）运算符* 是”间接值“，也叫作：解引用

参考代码：
```cpp
// pointer.cpp -- our first pointer variable
#include <iostream>
using namespace std;
int main()
{
    int updates = 6;        // declare a variable
    int * p_updates;        // declare pointer to an int
    p_updates = &updates;   // assign address of int to pointer
// express values two ways
    cout << "Values: updates = " << updates;
    cout << ", *p_updates = " << *p_updates << endl;
// express address two ways
    cout << "Addresses: &updates = " << &updates;
    cout << ", p_updates = " << p_updates << endl;
// use pointer to change value
    *p_updates = *p_updates + 1;
    cout << "Now updates = " << updates << endl;
    // cin.get();
    return 0;
}
```
![[Pasted image 20230726203248.png]]

（4）声明和初始化指针：
>【1】初始声明：
>>![[Pasted image 20230726214115.png]]
>>>==tips：对于每个指针变量名，都需要使用一个*==
```cpp
int* p1 , p2;    // 创建的是指针p1 和 int型p2

reason[系统自动识别过程]： int ——— *p1 ——— p2 
```

>【2】示例展示：
```cpp
// init_ptr.cpp -- initialize a pointer
#include <iostream>
int main()
{
    using namespace std;
    int higgens = 5;

    int * pt = &higgens;    // 被初始化的是指针pt，而不是它指向的值！
    cout << "Value of higgens = " << higgens
         << "; Address of higgens = " << &higgens << endl;
    cout << "Value of *pt = " << *pt
         << "; Value of pt = " << pt << endl;
    // cin.get();
    return 0;
}
```

（5）使用new来分配内存：
>>通用格式：typename * pointer_name = new typename;
>>eg:    int ****i = new int ;***
>>![[Pasted image 20230726214237.png]]

>>示例：
```cpp
// use_new.cpp -- using the new operator
#include <iostream>
using namespace std;
int main() {
    int nights = 1001;
    
    int * pt = new int;         // allocate space for an int
    *pt = 1001;                 // store a value there
    cout << "nights value = ";
    cout << nights << ": location " << &nights << endl;
    cout << "int ";
    cout << "value = " << *pt << ": location = " << pt << endl;
    
    double * pd = new double;   // allocate space for a double
    *pd = 10000001.0;           // store a double there

    cout << "double ";
    cout << "value = " << *pd << ": location = " << pd << endl;
    cout << "location of pointer pd: " << &pd << endl;
    
    cout << "size of pt = " << sizeof(pt);
    cout << ": size of *pt = " << sizeof(*pt) << endl;
    cout << "size of pd = " << sizeof pd;
    cout << ": size of *pd = " << sizeof(*pd) << endl;
    // cin.get();
    return 0;
}

结果：
nights value = 1001: location 0x61fe14
int value = 1001: location = 0xfd1640
double value = 1e+07: location = 0xfd1660
location of pointer pd: 0x61fe08
size of pt = 8: size of *pt = 4
size of pd = 8: size of *pd = 8
```
ps：sizeof( p ) 与 sizeof( ****p*** ) 的区别：
[深入理解计算机各种类型大小（sizeof） - DoubleLi - 博客园 (cnblogs.com)](https://www.cnblogs.com/lidabo/p/3618345.html)

（6）使用delete来释放内存：
>>>它将在使用完内存后，将其归还给内存池；归还/释放的内存可供程序的其他部分使用
```cpp
int *ps = new int;
......
delete ps;

!!!这将释放ps指向的内存，但不会删除指针ps本身!!!
例如：可以将ps重新指向另一个新分配的内存块
```

1）注意new与delete的严格配对！
2）不要创建两个指向同一个内存块的指针，因为这样极大可能产生"double free"错误！

（7）使用new来创建并使用动态数组：
>[1] 通用格式：typename * pointer_name = new typename [ num ]  ;

>[2] 创建：
![[Pasted image 20230726213109.png]]
>[3] 使用:
![[Pasted image 20230726213452.png]]
>[4]示例：==要注意指针+/-的含义==
```cpp
// arraynew.cpp -- using the new operator for arrays
#include <iostream>
int main()
{
    using namespace std;
    double * p3 = new double [3]; // space for 3 doubles
    p3[0] = 0.2;                  // treat p3 like an array name
    p3[1] = 0.5;
    p3[2] = 0.8;

    cout << "p3[1] is " << p3[1] << ".\n";

    p3 = p3 + 1;                  // increment the pointer
    cout << "Now p3[0] is " << p3[0] << " and ";
    cout << "p3[1] is " << p3[1] << ".\n";

    p3 = p3 - 1;                  // point back to beginning
    delete [] p3;                 // free the memory
    // cin.get();
    return 0;
}

结果：
p3[1] is 0.5.
Now p3[0] is 0.5 and p3[1] is 0.8.
```
![[Pasted image 20230726213922.png]]

[5]指针小结：
![[Pasted image 20230726215756.png]]
![[Pasted image 20230726215816.png]]

### 8) 指针、数组、指针运算：
>(1)指针与字符串：
>![[Pasted image 20230726220411.png]]
>==char 数组名、char指针、"str......"都被解释为：str第一个字符的地址==

```cpp
// ptrstr.cpp -- using pointers to strings
#include <iostream>
#include <cstring>              // declare strlen(), strcpy()
int main()
{
    using namespace std;
    char animal[20] = "bear";   // animal holds bear
    const char * bird = "wren"; // bird holds address of string
    char * ps;                  // uninitialized

1)  char字符串的输出只需要数组名：
    
    cout << animal << " and ";  // display bear[通过数组名输出整体str]
    cout << bird << "\n";       // display wren[通过数组名输出整体str]

2)  通过地址链接：

    cout << "Enter a kind of animal: ";
    cin >> animal;              // ok if input < 20 chars
    ps = animal;                ---------!!! set ps to point to string
    cout << ps << "!\n";       // ok, same as using animal
    cout << "Before using strcpy():\n";
    cout << animal << " at " << (int *) animal << endl;
    cout << ps << " at " << (int *) ps << endl;
    
3） 指针指向内存重覆盖：
    ps = new char[strlen(animal) + 1];  // get new storage
    strcpy(ps, animal);         // copy string to new storage
    cout << "After using strcpy():\n";
    cout << animal << " at " << (int *) animal << endl;
    cout << ps << " at " << (int *) ps << endl;

    delete [] ps;
    // cin.get();
    // cin.get();
    return 0;
}

结果：
bear and wren
Enter a kind of animal: hbx
hbx!
Before using strcpy():
hbx at 0x61fdf0
hbx at 0x61fdf0
After using strcpy():
hbx at 0x61fdf0
hbx at 0xea1630
```

**==程序说明：**
1）一般来说，给cout提供一个指针，它将打印地址
2）但是如果指针类型是 char* ，则cout将显示指向的字符串 [识别它为数组名]
3）如果要显示字符串地址，必须进行强制转换！eg:    (int*) ps
4）将animal赋给ps只是复制地址，这两个指针指向相同的内存单元[装有str]；而不是复制字符串==

**字符串函数strcpy(... , ... , ...) 的使用简介：**
![[Pasted image 20230726222806.png]]
![[Pasted image 20230726222938.png]]


（2）使用new创建动态结构：
访问方法：
>1）p->member
>![[Pasted image 20230726223453.png]]
>2）(****p***).member
>>>![[Pasted image 20230726223610.png]]
```cpp
// newstrct.cpp -- using new with a structure
#include <iostream>
using namespace std;
struct inflatable {   // structure definition
    char name[20];
    float volume;
    double price;
};

int main()
{
    inflatable * ps = new inflatable; // allot memory for structure
    
    cout << "Enter name of inflatable item: ";
    cin.get(ps->name, 20);            // method 1 for member access
    cout << "Enter volume in cubic feet: ";

    cin >> (*ps).volume;              // method 2 for member access
    cout << "Enter price: $";

    cin >> ps->price;
    
    cout << "Name: " << (*ps).name << endl;              // method 2
    cout << "Volume: " << ps->volume << " cubic feet\n"; // method 1
    cout << "Price: $" << ps->price << endl;             // method 1

    delete ps;                        // free memory used by structure
    // cin.get();
    // cin.get();
    return 0;
}
```

### 9）类型组合实践：
```cpp
// mixtypes.cpp --some type combinations
#include <iostream>
struct antarctica_years_end {
    int year;
};

int main()
{
    //1)创建结构体类型变量：
    antarctica_years_end s01, s02, s03;
    //2)采用成员运算符访问成员：
    s01.year = 1998;
    //3)创建指向这种类型的指针：
    antarctica_years_end * pa = &s02;
    //4)指针设置为有效地址后，解引用访问成员
    pa->year = 1999;
    
    //5)创建结构体数组：
    antarctica_years_end trio[3]; // array of 3 structures
    trio[0].year = 2003;
    std::cout << trio->year << std::endl; // 数组名=>首地址=>trio[0]的值
    //需要注意的是：数组名本身就是一个指针！！！！！！
    //同理：trio[1].year = 2004   =>  (trio+1)->year = 2004
    
    //6)元素为指针的数组 + 指向指针的指针
    const antarctica_years_end * arp[3] = {&s01, &s02, &s03};//内含三个指针的数组
    std::cout << arp[1]->year << std::endl;   //对应：s02指针所指向的内存中的某成员

    const antarctica_years_end ** ppa = arp;  //创建指向上数组的指针
    //arp是一个数组的名称，因此它是首元素的地址，因此ppa是一个指针，指向的是：一个指向antarctica_years_end的指针
    auto ppb = arp; // C++0x automatic type deduction
// or else use const antarctica_years_end ** ppb = arp;
    //7)ppa是指向指针的指针，因此*ppa是一个结构指针
    std::cout << (*ppa)->year << std::endl;//因此(*ppa)->year是s01的成员
    std::cout << (*(ppb+1))->year << std::endl;
    // std::cin.get();
    return 0;
}

结果：
2003
1999
1998
1999
```
