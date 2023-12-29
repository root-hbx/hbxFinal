# 考前小练习
>made by hbx

### 编辑距离（dp）
```cpp
#include <algorithm>
#include <cstring>
#include <iostream>
using namespace std;
const int N = 1e1 + 5, M = 1e3 + 10;

int n, m;
char str[M][N];
int dp[N][N];

int edit_distance(char a[], char b[])
{
    int la = strlen(a + 1), lb = strlen(b + 1);

    for (int i = 0; i <= lb; i++) {
        dp[0][i] = i;
    }
    for (int i = 0; i <= la; i++) {
        dp[i][0] = i;
    }

    for (int i = 1; i <= la; i++) 
    {
        for (int j = 1; j <= lb; j++) 
        {
            dp[i][j] = min(dp[i - 1][j] + 1, dp[i][j - 1] + 1);
            dp[i][j] = min(dp[i][j], dp[i - 1][j - 1] + (a[i] != b[j]));
        }
    }
    return dp[la][lb];
}

int main()
{
    cin >> n >> m;
    for (int i = 0; i < n; i++) {
        cin >> (str[i] + 1);
    }

    while (m--) 
    {
        int res = 0;
        char s[N];
        int limit;
        cin >> (s + 1) >> limit;
        for (int i = 0; i < n; i++) {
            if (edit_distance(str[i], s) <= limit) {
                res++;
            }
        }
        cout << res << endl;
    }

    return 0;
}
```

### 最短编辑距离（dp）
```cpp
#include<iostream>
#include<algorithm>
using namespace std;
const int maxn = 1e3+9;
int n,m;
char a[maxn],b[maxn];
int f[maxn][maxn];
/*
    f[i][j]: 将a[1~i] => b[1~j] 操作次数的min
    回看一步：f[i][j]从何而来？
    【根据相对性，这里我们以a字符串作为考虑主体】
    1) 删除了，导致完全对应:说明原状态是[i-1] <=> [j]   => f[i][j] = f[i-1][j] + 1
    2) 增加了，导致完全对应:说明原状态是[i] <=> [j-1]   => f[i][j] = f[i][j-1] + 1
    3) 替换了，导致完全对应:说明原状态是[i-1] <=> [j-1] => f[i][j] = f[i-1][j-1] + 1/0(取决于: a[i]==b[j]?)

*/

int main()
{
    cin >> n >> (a+1);
    cin >> m >> (b+1);
    // 1) 明确如何边界初始化！
    for(int i=1;i<=m;i++) f[0][i] = i; 
    for(int i=1;i<=n;i++) f[i][0] = i;
    
    // 2) f[i][j]动态规划：
    for(int i=1;i<=n;i++)
    {
        for(int j=1;j<=m;j++)
        {
            // f[i][j] = min(f[i-1][j] + 1, f[i][j-1] + 1);
            // if(a[i] == b[j]) {
            //     f[i][j] = min(f[i][j], f[i-1][j-1]);
            // }
            // else {
            //     f[i][j] = min(f[i][j], f[i-1][j-1] + 1);
            // }
            
            // 这里也可换一种写法：
            // 在C++中，std::min 函数接受两个参数
            f[i][j] = min(f[i-1][j]+1, f[i][j-1]+1);
            f[i][j] = min(f[i][j] , f[i-1][j-1]+(!(a[i]==b[j])));
        }
    }

    cout << f[n][m] <<endl;
    return 0;
}
```

### 最长上升子序列（dp实现）
```cpp
#include <iostream>
#include <algorithm>
using namespace std;
const int maxn = 1e3+9;
int n;
int a[maxn];
int f[maxn];
/*
    f[i]: 以i结尾的“最长上升子序列”的长度
    很显然：必然包含的是a[i]
    那么我们需要考虑f[i]的来源：
    在1~(i-1)所对应的基础上，a[i]可以加入其中，形成新的max_len
    故：f[i] = max(f[j]+1)，其中：j = 0 ~ (i-1)，且需要：a[j] < a[i]
*/

int main()
{
    cin >> n;
    for(int i=1;i<=n;i++) cin >> a[i];

    for(int i=1;i<=n;i++)
    {
        f[i] = 1;
        for(int j=1;j<=i-1;j++)
        {
            if(a[j] < a[i]) {
                f[i] = max(f[i] , f[j] + 1);
            }
        }
    }

    int res = 1;
    for(int i=1;i<=n;i++) res = max(res , f[i]);
    cout<<res<<endl;
    return 0;
}
```

### 滑雪（记忆化搜索）
```cpp
#include<iostream>
#include<algorithm>
#include<cstring>
using namespace std;
const int maxn = 305;

int a[maxn][maxn];
int f[maxn][maxn];
int r,c;
int dx[4] = {-1,0,0,1};
int dy[4] = {0,1,-1,0};
/*
    f[i][j]:
    从点(i,j)出发的路径长度max
*/

int dp(int x,int y)
{
    if(f[x][y] != -1) return f[x][y];
    
    f[x][y] = 1;
    
    for(int i=0;i<4;i++)
    {
        int xx = x + dx[i];
        int yy = y + dy[i];
        if(xx>=1 and xx<=r and yy>=1 and yy<=c)
        {
            if(a[x][y] > a[xx][yy]) {
                f[x][y] = max(f[x][y] , dp(xx,yy) + 1);
            }
        }
    }
    return f[x][y];
}

int main()
{
    cin >> r >> c;
    for(int i=1;i<=r;i++) {
        for(int j=1;j<=c;j++) {
            cin >> a[i][j];
        }
    }

    memset(f,-1,sizeof f);

    int res = 0;
    for(int i=1;i<=r;i++)
    {
        for(int j=1;j<=c;j++)
        {
            res = max(res,dp(i,j));
        }
    }

    cout << res <<endl;
    return 0;
}
```
