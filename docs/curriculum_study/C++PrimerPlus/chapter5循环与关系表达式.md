### 1）for 循环：
（1）组成部分：
```cpp
for(initialization; test-expression ; update-expression)
{
    body...
}
```

（2）设置步骤：
>[1]设置初始值
>[2]执行测试，考察循环是否可以继续执行
>[3]执行循环
>[4]更新用于测试的值

（3）要点说明：
>[1]test-expression决定循环体是否被执行，这个表达式是关系表达式，值为0的表达式被转化成bool值false，导致循环结束；反之继续！
```cpp
eg：for ( int i = limit ; i ; i-- )   // 当 i 减为 0 时退出循环
```

>[2]update-expression在每轮循环结束时执行，此时循环体已经执行完毕
```cpp
eg: for( int i = 1 ; i <= 100 ; i += 2 )  // 每次 + 2
eg: for( int i = 0 ; i <= 1000 ; i = i*i + 3 )
```

>[3]自增自减表达式：
>x++:  使用x的当前值计算表达式，然后将x的值+1
>++x:  先将x的值+1，然后用新的值计算表达式
>x--:    同上
>--x:    同上

>[4]前/后缀格式在for循环中的异同：
```cpp
下面二者相同 [reason：不管是--n还是n--，它们都将在程序进入下一步之前完成，终效果相同]
	for(int n=lim;n;--n){......}    // 前缀的效率更高

	for(int n=lim;n;n--){......}    // 后缀的效率相对低
```

（4）工具：
>1）递增/递减运算符 与 指针：
>[1]前缀 递增/递减 and 解引用 优先级相同，利用自右向左的方式结合
>[2]后缀 递增/递减 优先级相同，但比前缀运算符的优先级高，这两个运算符利用自左向右的方式结合
>![[Pasted image 20230727092754.png]]
>![[Pasted image 20230727092811.png]]


>2）逗号表达式：
>[1]常见应用：允许把两个表达式放到“只允许放一个表达式”的地方
```cpp
for(int j = 0,i = word.size() - 1 ; j < i ; --i,++j)
{
	......
}
```
>
>[2]自身性质：
>（1）它先计算第一个表达式，而后计算第二个表达式
```cpp
i = 20 , j = 2 * i;    // i set to 20 , then j ste to 40
```

>（2）逗号表达式的值是第二部分的值
>比如上述表达式整体的值是40  ( 因为 " , " 后面部分的值是40 )
>
>（3）在所有运算符中，逗号运算符的优先级是最低的
```cpp
eg1:    cats = 17 , 240;
被解释为：(cats=17),240;    将cats设置成17, 240不起作用！

eg2:    cats = (17 , 240);
因为括号的优先级最高，执行顺序是：
（1）括号内部：17,240    得出值是240
（2）240返回给括号
（3）括号通过'='向左赋值给cats
最终：将cats设置成 240！
```


### 2）while 循环：
（1）通用格式：
```cpp
while(test-condition)
{
    body...
}
```

（2）执行原理：
![[Pasted image 20230727094642.png]]

（3）设置类型别名：
方法1：通过预处理器
```cpp
#define BYTE char
//预处理器在编译程序时用char替换所有的BYTE，从而使BYTE成为char的别名
```

方法2：使用c++关键字 typedef 创建别名
```cpp
typedef char BYTE;
//使 BYTE 成为 char 的别名
```


### 3）do-while 循环：
通用格式：
```cpp
do
{
    ......
} while(test-expression)p;
```
执行原理：
![[Pasted image 20230727095957.png]]
>>>>>>>>>>>>>>>>>>>>![[Pasted image 20230727100011.png]]

### 4）基于范围的 for 循环：
![[Pasted image 20230727100318.png]]

### 5）循环与文本输入：
（1）cin 与 cin.get(ch) :
>[1]提出问题：
```cpp
// textin1.cpp -- reading chars with a while loop
#include <iostream>
int main()
{
    using namespace std;
    char ch;
    int count = 0;      // use basic input
    cout << "Enter characters; enter # to quit:\n";
    cin >> ch;          // get a character
    while (ch != '#')   // test the character
    {
        cout << ch;     // echo the character
        ++count;        // count the character
        cin >> ch;      // get the next character
    }
    cout << endl << count << " characters read\n";
    return 0;
}
```
![[Pasted image 20230727103027.png]]
![[Pasted image 20230727103043.png]]

[2]改进方法，并提出使用cin.get(ch)解决问题：
>方法：
```cpp
char ch;       //  不可以省略，否则就是未定义变量ch
cin.get(ch);   //  读取输入中的下一个字符(即使它是空格)，然后将其赋给ch
```
>例证：
```cpp
// textin2.cpp -- using cin.get(char)
#include <iostream>
int main()
{
    using namespace std;
    char ch;
    int count = 0;
    cout << "Enter characters; enter # to quit:\n";

    cin.get(ch);        // use the cin.get(ch) function
    while (ch != '#')
    {
        cout << ch;
        ++count;
        cin.get(ch);    // use it again
    }
    cout << endl << count << " characters read\n";
    return 0;
}
```

（2）文件尾（EOF）条件：
>原理：如果检测到文件尾的EOF，fail( )成员函数返回true；否则返回false！
```cpp
// textin3.cpp -- reading chars to end of file
#include <iostream>
using namespace std;
int main()
{
    char ch;
    int count = 0;
    cin.get(ch);        // attempt to read a char
    while (cin.fail() == false)  // test for EOF
    {
        cout << ch;     // echo character
        ++count;
        cin.get(ch);    // attempt to read another char
    }
    cout << endl << count << " characters read\n";
    return 0;
}
```
ps： windows系统上以ctrl+Z模拟EOF条件


### 6）二维数组与嵌套循环：

>过于简单，略之！