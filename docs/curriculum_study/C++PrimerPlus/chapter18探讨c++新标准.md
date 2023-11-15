##### 1）c++11的新功能
（0）新类型：新增了long long 和 unsigned long long

（1）统一的初始化：
![[Pasted image 20230807222146.png]]

（2）缩窄：
![[Pasted image 20230807222316.png]]
【1】允许往宽转
【2】允许（在合理范围内）向窄转

（3）声明：
【1】auto：实现自动类型推断and简化模板声明
![[Pasted image 20230807222441.png]]

【2】decltype：抽象意义上将变量的类型同化
![[Pasted image 20230807222600.png]]
![[Pasted image 20230807223008.png]]
[【C语言】深入理解C语言中的整型提升和算术转换_宁清_的博客-CSDN博客](https://blog.csdn.net/m0_73898917/article/details/129769162)

（4）返回类型后置：
![[Pasted image 20230807223112.png]]

（5）nullptr：
[1] 空指针是不会指向有效数据的指针
[2] nullptr\==0 为true
[3] 可将0传递给接受int参数的函数，但是若将nullptr传递之，则会报：编译错误

（6）对类的修改
【1】explicit：禁止构造函数、转换函数进行自动转换
![[Pasted image 20230807224033.png]]
![[Pasted image 20230807224043.png]]

【2】==可以进行类内成员初始化==
![[Pasted image 20230807224248.png]]
![[Pasted image 20230807224300.png]]

得出结论：
>对于对象的调用，实参 > 形参(初始化列表) > 类内初始化成员

（7）右值引用
![[Pasted image 20230807225326.png]]
![[Pasted image 20230807225337.png]]

##### 2）移动语义和右值引用
![[Pasted image 20230807225649.png]]

赋值过程解析：
![[Pasted image 20230807230039.png]]


##### 3）Lambda函数（Lambda表达式）
（1）问题引入：
常规操作：
![[Pasted image 20230807230917.png]]
![[Pasted image 20230807230940.png]]
![[Pasted image 20230807230952.png]]

---
使用Lambda函数的操作：
![[Pasted image 20230807231136.png]]
![[Pasted image 20230807231146.png]]

（2）使用Lambda函数的优点：
【1】
![[Pasted image 20230807231348.png]]

【2】
![[Pasted image 20230807231403.png]]

【3】
![[Pasted image 20230807231523.png]]

---
## ps：Lambda 函数与表达式

C++11 提供了对匿名函数的支持,称为 Lambda 函数(也叫 Lambda 表达式)。

Lambda 表达式==把函数看作对象==。Lambda 表达式可以像对象一样使用，比如==可以将它们赋给变量和作为参数传递==，还==可以像函数一样对其求值。==


（1）Lambda 表达式本质上与函数声明非常类似。Lambda 表达式具体形式如下:
```cpp
[capture](parameters)->return-type{body}
```
例如：
\[ ](int x, int y){ return x < y ; }

（2）如果没有返回值可以表示为：
```cpp
[capture](parameters){body}
```
如果 lambda 函数没有传回值（例如 void），其返回类型可被完全忽略。

例如：
\[ ]{ ++global_x; } 

（3）在一个更为复杂的例子中，返回类型可以被明确的指定如下：
\[ ](int x, int y) -> int { int z = x + y; return z + x; }
本例中，一个临时的参数 z 被创建用来存储中间结果。如*同一般的函数，z 的值不会保留到下一次该不具名函数再次被调用时*。


（4）在Lambda表达式内可以访问当前作用域的变量，这是Lambda表达式的闭包（Closure）行为。 与JavaScript闭包不同，C++变量传递有传值和传引用的区别。可以通过前面的\[ ]来指定：

\[]      // 沒有定义任何变量。使用未定义变量会引发错误。
\[x, &y] // x以传值方式传入（默认），y以引用方式传入。
\[&]     // 任何被使用到的外部变量都隐式地以引用方式加以引用。
\[=]     // 任何被使用到的外部变量都隐式地以传值方式加以引用。
\[&, x]  // x显式地以传值方式加以引用。其余变量以引用方式加以引用。
\[=, &z] // z显式地以引用方式加以引用。其余变量以传值方式加以引用。

另外有一点需要注意。对于[=]或[&]的形式，lambda 表达式可以直接使用 this 指针。但是，对于[]的形式，如果要使用 this 指针，必须显式传入：
[this]() { this->someFunc(); }();