### 1）climits中的符号常量：
![[Pasted image 20230726105247.png]]
```cpp
#include <iostream>
#include <climits>
using namespace std;
int main(int argc, char const *argv[])
{

    int n_int = INT_MAX;
    
    short n_short = SHRT_MAX;
    
    long n_long = LONG_MAX;
    
    long long n_llong = LLONG_MAX;
    
    std::cout << "int is       " << CHAR_BIT << "  bytes." << '\n';

    std::cout << "short is     " << sizeof(short) << "  bytes." << '\n';

    std::cout << "long is      " << sizeof(long) << "  bytes." << '\n';

    std::cout << "long long is " << sizeof(long long) << "  bytes." << '\n';

    std::cout << "Maxinum values:" << '\n';

    std::cout << "int       : " << n_int << '\n';

    std::cout << "short     : " << n_short << '\n';

    std::cout << "long      : " << n_long << '\n';

    std::cout << "long long : " << n_llong << '\n';

    std::cout << "Bits per byte : " << CHAR_BIT << '\n';

    return 0;

}
-----------------------------------------------------------------------------------
结果：

int is       8  bytes.
short is     2  bytes.
long is      4  bytes.
long long is 8  bytes.
Maxinum values:
int       : 2147483647
short     : 32767
long      : 2147483647
long long : 9223372036854775807
Bits per byte : 8
```

### 2）ASCII 表：
![[Pasted image 20230726105531.png]]

### 3）缩放因子表示方法：AeB / AEB   
eg: 1e6 = 10^6      5.98E24 = 5.98 x 10^24     9.11e-34 
（1）指数可以是正数，也可以是负数
（2）E与e在此处等价
（3）数字中不可以有空格  eg： 7.2  e6   非法

### 4）c++11中的auto声明：  自动推断变量的类型
例如：
```cpp
auto n = 100;       // n is int 
auto x = 1.5;       // x is double
auto y = 1.3e12L;   // y is long double
```

auto的真正优势在处理复杂类型中得以体现，比如STL中的类型：
```cpp
[approach1]
std::vector<double> hbx;
std::vector<double>::iterator hhh = hbx.begin();

[approach2]
std::vector<double> hbx;
auto hhh = hbx.begin();

对比可知：
std::vector<double>::iterator = auto
```