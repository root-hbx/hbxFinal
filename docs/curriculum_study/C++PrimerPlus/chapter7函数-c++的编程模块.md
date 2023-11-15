### （1）基本知识：
>略之！

### （2）函数指针：
>这里只介绍基础知识，高级知识现在没法理解！
>[1]引入;
>![[Pasted image 20230727113331.png]]
>
>[2]基础知识：
>（1）要点：获取函数的地址 + 声明一个函数指针 + 使用函数指针调用函数
>
>（2）获取函数的地址：
>>方法：只需要使用函数名（后面不跟参数）即可
>>eg：如果think()是一个函数，则think就是函数地址
>>==***要将函数作为参数进行传递，必须传递函数名！***==
>
>注意区分传递的是 函数的地址 还是 函数的返回值 ！
```cpp
已有函数think():
[1] process( think )    // process()函数能在内部调用think()函数

[2] thought( think() )  // thought()首先调用think()函数，然后将think()返回值传递给                              thought()函数
```

>（3）声明函数指针：
>声明应当指定：函数的返回类型 + 函数的参数列表
```cpp
（1）已有函数： double pam(int);
（2）建立正确的函数指针声明：
	double (*pf)(int);
	//1)  (*pf)与函数名等价，pf是指向pam函数的指针
	//2)  指针pf指向的函数pam的参数类型是int，返回值是double  
（3）正确声明pf之后，可以将相应的函数地址赋给它：
	double pam(int);
	double (*pf)(int);
	pf = pam;         // pf now points to the pam() function

（4）使用过程：
	void estimate( int lines , double (*pf)(int) ) {...}
	......
	estimete(666,pam);   // function call telling estimate() to use pam()
	
```
ps: 1) 类型必须完全一一对应！    2) 由于运算符优先级，****pf外面的( )不可以省略！***
 
>（4）使用指针调用函数：
```cpp
double pam(int);
double (*pf)(int);
pf = pam;
...
double x = pam(4);
double y = (*pf)(7);
```

ps:
![[Pasted image 20230727115822.png]]
