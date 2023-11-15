### 1）c++对于大小写敏感:
    比如cout改写成Cout就没法通过编译

### 2）基础程序组成：
```cpp
<structure>
---------------------------------
注释：  由前缀 // 标识 
预处理器编译指令：#include<...>
函数头：int main()
编译指令： using namespace ...
函数体：   {......}
cout工具显示消息的语句：
结束 main() 函数的return语句：
```
### 3）函数头：int main()
		(1)   描述的是main()函数与操作系统之间的接口
		(2)   空括号means：main()函数不接受任何信息/不接受任何参数
		(3)   tips：让括号空着意味着对于是否接受参数保持沉默

### 4）c++预处理器与iostream文件：
		(1) # include<iostrteam>    导致预处理器将iostream文件中的内容添加到程序中【在源代码被编译之前，替换或者添加文本】
		(2) 像iostream这样的文件叫做包含文件(include file) 也叫作头文件(head file)

### 5）名称空间 ：using namespace ...
			(1) 这叫做using编译指令
			(2) using namespace std;  使得std名称空间中的所有名称都可以使用
			(3) 勤劳写法：using std::cout;        using std::endl;     ......

### 6）程序实例:
```cpp
// carrots.cpp -- food processing program
// uses and displays a variable

#include <iostream>
int main()
{
    using namespace std;
    int carrots;            // declare an integer variable
    carrots = 25;            // assign a value to the variable
    cout << "I have ";
    cout << carrots;        // display the value of the variable
    cout << " carrots.";
    cout << endl;
    carrots = carrots - 1;  // modify the variable
    cout << "Crunch, crunch. Now I have " << carrots << " carrots." << endl;
    // cin.get();

    return 0;
}
```