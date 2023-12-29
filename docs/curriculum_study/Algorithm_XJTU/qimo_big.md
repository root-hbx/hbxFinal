# 常考算法模板
>organized by hbx

### 快速排序
```cpp
#include<bits/stdc++.h>
#define maxn 100009
using namespace std;
int n;
int a[maxn];

void sort_quick(int a[],int l,int r)
{
    if(l >= r) return;

    int i = l - 1,j = r + 1;//定下边界（注意：需要留空！一个-1 、一个+1）

    int x = a[(l+r)>>1];//选定基数

    while(i < j)
    {
        do(i++);while(a[i] < x);
        do(j--);while(a[j] > x);
        if(i < j) {
            swap(a[i],a[j]);
        }
    }

    sort_quick(a,l,j);    
    sort_quick(a,j+1,r);
/*
注意·此处写法的对称性：
1) 如果前面的基数选定的是x = a[l],则必须是(a,l,j)与(a,j+1,r),不可以是(a,l,i-1)与(a,i,r)
2) 如果前面的基数选定的是x = a[r],则必须是(a,l,i-1)与(a,i,r),不可以是(a,l,j)与(a,j+1,r)
3) 一般来说更推荐选定基数是x = a[(l+r)/2]
*/
}

int main()
{
    cin >> n;
    for(int i=1;i<=n;i++) cin >> a[i];
    
    sort_quick(a,1,n);

    for(int i=1;i<=n;i++) cout << a[i] <<" ";
    cout << endl;
    return 0;
}
```

最佳时间复杂度 O($n^2$)
最坏时间复杂度 O(nlogn)
平均时间复杂度 O(nlogn)
不稳定
### 归并排序
```cpp
#include<iostream>
#include<algorithm>
#define maxn 1000009
using namespace std;

int a[maxn];
int tmp[maxn];

void merge_sort(int a[], int l, int r)
{
    if(l >= r) return;
    
    int mid = (l+r)/2;
    merge_sort(a,l,mid);
    merge_sort(a,mid+1,r);

    int i = l , j = mid + 1;
    int k = 0;

    while(i<=mid and j<=r)
    {
        if(a[i] <= a[j]) tmp[k++] = a[i++];
        else tmp[k++] = a[j++];
    }

    while(i<=mid) tmp[k++] = a[i++];
    while(j<=r) tmp[k++] = a[j++];

    for(int i=l,j=0; i<=r; i++,j++) a[i] = tmp[j];
}

int main()
{
    int n;
    cin >> n;
    for(int i=1;i<=n;i++) cin >> a[i];

    merge_sort(a,1,n);
  
    for(int i=1;i<=n;i++) cout << a[i] << ' ';
    cout<<endl;
    return 0;
}
```

平均时间复杂度 O(nlogn)
稳定
### 最长公共子序列
![[Pasted image 20231224230420.png]]

```cpp
#include<iostream>
#include<algorithm>
#define maxn 1009
using namespace std;

int n,m;
char a[maxn],b[maxn];
int f[maxn][maxn];

int main()
{
    cin >> n >> m;
    cin >> a+1 >> b+1;

    for(int i=1; i<=n; i++)
    {
        for(int j=1; j<=m; j++)
        {
            f[i][j] = max(f[i-1][j],f[i][j-1]);
            
            if(a[i] == b[j]) {
                f[i][j] = max(f[i][j],f[i-1][j-1]+1);
            }
        }
    }

    cout << f[n][m];

    return 0;
}
```
### 0-1背包问题（dp/贪心/回溯/分支限界）
```cpp
/* 动态规划方法
0-1背包：
def: 
    f[i][j]: 只看前i个物品，总体积是j的情况下，总价值max是多少？
ans:
    res = max{f[n][0~v]}
ana:
    f[i][j] = 出发点是看物品i是否被选
    1）不选物品i，f[i][j] = f[i-1][j]
    2）选物品i，f[i][j] = f[i-1][j-v[i]]
    取二者max即可
ori:
    f[0][0] = 0
flex:
    O(nm)
*/

#include<iostream>
#include<algorithm>
#define maxn 1009
using namespace std;

int n,m;
int f[maxn][maxn];
int v[maxn],w[maxn];

int main()
{
    cin >> n >> m;
    for(int i=1;i<=n;i++) cin >> v[i] >> w[i];
    
    f[0][0] = 0;

    for(int i=1;i<=n;i++)
    {
        for(int j=0;j<=m;j++) // 注意这里体积枚举需从0开始
        {
            f[i][j] = f[i-1][j];

            if(j>=v[i]) {
                f[i][j] = max(f[i-1][j] , f[i-1][j-v[i]] + w[i]);   
            }
        }
    }
    
    int res = 0;
    for(int i=0;i<=m;i++) res = max(res,f[n][i]);
    cout<<res<<endl;
    return 0;
}
```

```cpp
/*
贪心方法：
贪心算法是一种只考虑当前最优的算法，其不从总体上考虑
所以贪心算法不是对所有问题都能求得整体最优解！像0-1背包问题，用贪心算法一般求得的是局部最优解

用贪心算法解0-1背包问题原理如下：
首先我们按照物品的单位重量价值来进行排序，然后按照单位重量价值从高到低依次进行选择，若其能装入背包则将其装入，不能则继续判断下一个直至所有物品都判断完，就得到了问题的一个解。

但是对于0-1背包问题，用贪心算法并不能保证最终可以将背包装满，部分剩余的空间使得单位重量背包空间的价值降低了，这也是用贪心算法一般无法求得0-1背包最优解的原因。
*/

#include <iostream>
#include<algorithm>
using namespace std;

int n, c; //n个物品，c的容量

struct node
{
	double v;//价值
	double w;//重量
} wu[100];

bool cmp1(node a, node b)//重量
{
	if (a.w == b.w)
		return a.v > b.v;
	return a.w < b.w;
}

bool cmp2(node a, node b)//价值
{
	if (a.v == b.v)
		return a.w < b.w;
	return a.v > b.v;
}

bool cmp3(node a, node b)// 单位价值
{
	if ((a.v / a.w) == (b.v / b.w))
		return a.w < b.w;
	return (a.v / a.w) > (b.v / b.w);
}

int fun1(int n, int c)
{
	sort(wu, wu + n, cmp1);
	int value = 0;
	for (int i = 0; i < n; i++)
	{
		if (c >= wu[i].w)
		{
			c -= wu[i].w;
			value += wu[i].v;
		}
	}
	return value;
}

int fun2(int n, int c)
{
	sort(wu, wu + n, cmp2);
	int value = 0;
	for (int i = 0; i < n; i++)
	{
		if (c >= wu[i].w)
		{
			c -= wu[i].w;
			value += wu[i].v;
		}
	}
	return value;
}

int fun3(int n, int c)            // 这才是本题贪心所对应的解法！
{
	sort(wu, wu + n, cmp3);
	int value = 0;
	for (int i = 0; i < n; i++)
	{
		if (c >= wu[i].w)
		{
			c -= wu[i].w;
			value += wu[i].v;
		}
	}
	return value;
}

int main()
{
	cin >> n >> c;
    for (int i = 0; i < n; i++) cin >> wu[i].w;
	for (int j = 0; j < n; j++) cin >> wu[j].v;
	
    cout << "优先放重量最小的答案：";
	cout << fun1(n, c) << endl;

	cout << "优先放价值最大的答案：";
	cout << fun2(n, c) << endl;
	
    cout << "先放性价比最大的答案：";
	cout << fun3(n, c) << endl;
	
	return 0;
}
```

```cpp
/* https://blog.csdn.net/qq_67764134/article/details/129903263
回溯法：
1) 01背包属于找最优解问题，用回溯法需要构造解的子集树。对于每一个物品i，对于该物品只有选与不选2个决策，总共有n个物品，可以顺序依次考虑每个物品，这样就形成了一棵解空间树： 基本思想就是遍历这棵树，以枚举所有情况，最后进行判断，如果重量不超过背包容量，且价值最大的话，该方案就是最后的答案。
2) 在搜索状态空间树时，只要左子节点是可一个可行结点，搜索就进入其左子树。对于右子树时，先计算上界函数，以判断是否将其减去（剪枝）。
上界函数bound():当前价值cw+剩余容量可容纳的最大价值<=当前最优价值bestp。　
3) 为了更好地计算和运用上界函数剪枝，选择先将物品按照其单位重量价值从大到小排序，此后就按照顺序考虑各个物品!!!!!!
4) 在递归函数Backtrack中，
　　当i>n时，算法搜索至叶子结点，得到一个新的物品装包方案。此时算法适时更新当前的最优价值
　　当i<n时，当前扩展结点位于排列树的第（i-1）层，此时算法选择下一个要安排的物品，以深度优先方式递归的对相应的子树进行搜索，对不满足上界约束的结点，则剪去相应的子树。
*/

double Bound(int k)
{
    remain_v = 0;
    while(k <= n){
        remain_v += v[k];
        k++;
    }
    return remain_v + now_v;
}

void Backtrack(int t)
{
    if(t > n){  //是否到达叶节点
        for(int i = 1; i <= n; i++){
            best_x[i] = x[i];   //记录回溯的最优情况
        }
        best_v = now_v; //记录回溯中的最优价值
        return;
    }
    
    if(now_w + w[t] <= W){  //约束条件，是否放入。放入考虑左子树，否则考虑右子树
        x[t] = 1;
        now_w += w[t];
        now_v += v[t];
        Backtrack(t+1); //进行下一个节点的分析
        now_w -= w[t];  //在到达叶节点后进行回溯
        now_v -= v[t];
    }
    
    /*若对应的物品不能放入背包，继续分析“后面剩余所有物品”的价值加上当前的价值是否会大于我们前面求得的最优价值:
    若“后面剩余所有物品”的价值加上当前的价值小于，则我们就没必要在进行往下考虑。（最极限情况都不满足，那就没有接着考虑的必要了！）
    否则，我们需要继续往下考虑，即将当前的物品x[t]=0进行标记，接着对下一个物品调用回溯函数*/
    if(Bound(t+1) > best_v){ 
	//限界条件，是否剪枝。若放入t后不满足约束条件则进行到此处，然后判断若当前价值加剩余价值都达不到最优，则没必要进行下去
        x[t] = 0;
        Backtrack(t+1);
    }
}
```

```cpp
/* https://blog.csdn.net/qq_67764134/article/details/129903263
分支限界法：这里跟上述回溯区别不大！
*/
#include <iostream>
#define N 1009
using namespace std;
int n;double W; //n个数，W容量
double w[N];double v[N];  //物品重量和价值
bool x[N];
bool best_x[N]; //存储最优方案
double now_v;   //当前价值
double remain_v;    //剩余价值
double now_w;   //当前容量
double best_v;  //最优价值

double Bound(int k)     //计算分枝结点k的上界
{
    remain_v = 0;
    while (k <= n) {
        remain_v += v[k];
        k++;
    }
    return remain_v + now_v;
}
void Backtrack(int t)
{
    if (t > n) {  //是否到达叶节点
        for (int i = 1; i <= n; i++) {
            best_x[i] = x[i];   //记录回溯的最优情况
        }
        best_v = now_v; //记录回溯中的最优价值
        return;
    }
    if (now_w + w[t] <= W) {  //约束条件，是否放入。放入考虑左子树，否则考虑右子树
        x[t] = 1;
        now_w += w[t];
        now_v += v[t];
        Backtrack(t + 1); //进行下一个节点的分析
        now_w -= w[t];  //在到达叶节点后进行回溯
        now_v -= v[t];
    }
    if (Bound(t + 1) > best_v) {    //限界条件，是否剪枝。若放入t后不满足约束条件则进行到此处，然后判断若当前价值加剩余价值都达不到最优，则没必要进行下去
        x[t] = 0;
        Backtrack(t + 1);
    }
}
void Knapsack(double W, int n)
{
    double sum_w = 0;
    double sum_v = 0;
    best_v = 0;
    for (int i = 0; i < n; i++) {
        sum_w += w[i];
        sum_v += v[i];
    }
    Backtrack(1);
    cout << "放入购物车的物品最大价值为：" << best_v << endl;
}
int main()
{
    cout << "输入物品个数n:"; cin >> n;
    cout << "输入购物车容量W:"; cin >> W;
    cout << "依次输入每个物品的重量w和价值v，用空格分开：\n";
    for (int i = 1; i <= n; i++) {
        cin >> w[i] >> v[i];
    }
    Knapsack(W, n);
    return 0;
}
```
### 马的遍历
题目描述
有一个 $n \times m$ 的棋盘，在某个点 $(x, y)$ 上有一个马，要求你计算出马到达棋盘上任意一个点最少要走几步。
```cpp
#include <iostream>
#include<algorithm>
#include<queue>
#include<iomanip>
using namespace std;
struct pos { //一个结构体存储x,y两个坐标
    int x, y;
    pos(int ax = 0, int ay = 0) 
    {
        x = ax; 
        y = ay;
    }
};
const int maxn = 405;
queue<pos> Q;//队列
int ans[maxn][maxn];//记录答案，-1表示未访问
int n, m, sx, sy;
int walk[8][2] = {{2, 1}, {1, 2}, {-1, 2}, {-2, 1},
    {-2, -1}, {-1, -2}, {1, -2}, {2, -1}
};

int main()
{
    cin >> n >> m >> sx >> sy;
    for(int i = 1; i <= n; i++)
        for(int j = 1; j <= m; j++)
            ans[i][j] = -1; // 答案都初始化为未访问过
    Q.push(pos(sx, sy)); // 使起点入队扩展
    ans[sx][sy] = 0;//起点一定不能忘了“访问”这一步！！！

    while (!Q.empty()) { // 循环直到队列为空
        pos now = Q.front();  // 拿出队首以扩展
        Q.pop();
        for (int k = 0; k < 8; k++) {
            int x = now.x + walk[k][0]; 
            int y = now.y + walk[k][1]; // 计算新坐标 x 和 y
            int d = ans[now.x][now.y]; // d 是目前点的走几步的结果
            if (x < 1 || x > n || y < 1 || y > m || ans[x][y] != -1)
                continue;  // 若坐标超过地图范围或者该点已被访问过则无需入队
            ans[x][y] = d+1;  // 记录下一个点的答案，是目前点多走一步的结果。
            Q.push(pos(x, y));
        }
    }

    for(int i = 1; i <= n; i++){
        for(int j = 1; j <= m; j++)
            cout<<setw(5)<<ans[i][j]; // 输出第 i 行第 j 列答案，整数宽度 5 位
        puts(""); // 输出换行
    }
}
```
### n皇后问题
一个如下的 6×66×6 的跳棋棋盘，有六个棋子被放置在棋盘上，使得每行、每列有且只有一个，每条对角线（包括两条主对角线的所有平行线）上至多有一个棋子。

这只是棋子放置的一个解。请编一个程序找出所有棋子放置的解。  
并把它们以上面的序列方法输出，解按字典顺序排列。  
请输出前 3 个解。
最后一行是解的总个数。
```cpp
#include<bits/stdc++.h>
using namespace std;
//某一点一旦落定，它的"米"四相就已经落定!
int col[100],zd[100]/*主对角线*/,fd[100]/*副对角线*/; 
int sum;//解的总个数
int ctl_num = 0;//cmd =>控制输出前三个的数字 
int ans[100];//每一行的情况（答案数组，记录的是可取的col） 
int n;

//dfs第pos行的摆放方式（可取的col） 
void dfs(int pos)
{
	//0）达到目的地，输出结果： 
	if(pos==n+1)
	{
		if(ctl_num<3) {
			for(int i=1;i<=n;i++)
		    {
			    cout<<ans[i]<<" ";
		    }
		    cout<<endl;
		    
		    ctl_num++;
		    sum++;
	    }
		    
	    else {
	    	sum++;
		}
		return;
	}
	
	for(int i=1;i<=n;i++) //考察：第pos行摆在i列可不可以?! 
	{
		//1）异常情况排除： 
		if(col[i] or zd[n + (i-pos)] or fd[i+pos])
		{
			//第二个一定要注意补个n，不然下标会越界，极端情况是a[1-n]
			//由于我们数组下标从1开始，因此补 n即可 
			continue;
		}
		
		//2）正常递归情况：	
		col[i] = 1;
		zd[n + (i-pos)] = 1;
		fd[i+pos] = 1;
		ans[pos] = i;
		
		dfs(pos+1);
		
		//这次pos+1的寻找是建立在当前i的情况下的
		//后续可不是，因此需要撤销标记！回溯！ 
		col[i] = 0;
		zd[n + (i-pos)] = 0;
		fd[i+pos] = 0;
	}
} 

int main()
{
	cin >> n;
	//考察每一行可以在哪些位置放棋子
	//从第一行开始dfs："第一行"是本题设计的起点 
	dfs(1);
	cout<<sum<<endl;
	return 0;
}
```
### 布线问题
印刷电路板将布线区域划分成m\*n个方格阵列,如图(1)所示。  
精确的电路布线问题要求确定连接方格a的中点到方格b的中点的最短布线方案。  
在布线时,电路只能沿直线或直角布线,如图(2)所示。  
为了避免线路相交，已布了线的方格做了封锁标记,其他线路不允许穿过被封锁的方格
```cpp
bool IsEnd(node t){//判断到没到终点 
	if(t.x==end.x&& t.y==end.y) //到终点了 
		return true;
	else return false;
}

bool FindPath(){//广搜 
	if(IsEnd(start)) return true;//判断起点是不是等于终点
	
	queue<node>q;
	int newx,newy; node no;
	q.push(start);//起点入队 
	while(!q.empty() )
	{
		no = q.front(); q.pop();//取出队首 
		for(int i=0;i<4;i++) 
		{ //四个方向上下左右 
			newx = no.x + go[i][0];
			newy = no.y + go[i][1];
			
			if(grid[no.x][no.y]+1 < grid[newx][newy])
			{//剪枝 
				grid[newx][newy] = grid[no.x][no.y]+1;
				node t;
				t.x = newx;
				t.y = newy;
				if(IsEnd(t)) return true;//到终点了						
				else q.push(t);	//否则入队			
			}		
		}
	}
	return false; 
}
```
#### 类似题型：迷宫
题目描述
给定一个 $N \times M$ 方格的迷宫，迷宫里有 $T$ 处障碍，障碍处不可通过。
在迷宫中移动有上下左右四种方式，每次只能移动一个方格。数据保证起点上没有障碍。
给定起点坐标和终点坐标，每个方格最多经过一次，问有多少种从起点坐标到终点坐标的方案。

输入格式
第一行为三个正整数 $N,M,T$，分别表示迷宫的长宽和障碍总数。
第二行为四个正整数 $SX,SY,FX,FY$，$SX,SY$ 代表起点坐标，$FX,FY$ 代表终点坐标。
接下来 $T$ 行，每行两个正整数，表示障碍点的坐标。

输出格式
输出从起点坐标到终点坐标的方案总数。

样例输入 #1
```
2 2 1
1 1 2 2
1 2
```
样例输出 #1
```
1
```

```cpp
/*bfs版本*/
#include<cstdio>
#include<queue>
#include<cstring>
using namespace std;
int n,m,k,x,y,a,b,ans;
int dx[4] = {0,0,1,-1},dy[4] = {1,-1,0,0};
bool vis[6][6];
struct oo{
    int x,y,used[6][6];
};
oo sa;
void bfs()
{
    queue<oo> q;      //定义结构体后，可以直接使用结构体名定义变量或者队列。
    sa.x = x;
    sa.y = y;         //横纵坐标替换，这样写起来方便。
    sa.used[x][y] = 1;//标记走过的路径
    q.push(sa);
    while(!q.empty())
    {
        oo now = q.front();     //一起拿出来
        q.pop();
        for(int i = 0;i < 4; i++)
        {
            int sx = now.x + dx[i];
            int sy = now.y + dy[i];
            if( now.used[sx][sy] 
                || vis[sx][sy] 
                || sx == 0 || sy == 0 
                || sx > n || sy > m)
                continue;    //如果这里走过，或者这里是障碍，或者这里是墙壁，那么这里就不能走。
            if(sx == a && sy == b)
            {
                ans++;           //如果这里是终点，那么结果数量加一
                continue;
            }
            sa.x = sx;
            sa.y = sy;
            memcpy(sa.used,now.used,sizeof(now.used));
            sa.used[sx][sy] = 1;     //这里的操作都是为了标记路径
            q.push(sa);
        }
    }
}

int main()
{
    scanf("%d%d%d",&n,&m,&k);
    scanf("%d%d%d%d",&x,&y,&a,&b);         //虽然这里可以合并成一个句子，但是由于我是从python转过来的，建议大家以后写代码都设置一个界限，代码不宜太长
    for(int i = 1,aa,bb;i <= k; i++)     //大家注意一下，我现在是直接把变量定义在循环里面。
    {
        scanf("%d%d",&aa,&bb);           //现在我们不能走障碍了。
        vis[aa][bb] = 1;
    }
    bfs();
    printf("%d",ans);
    return 0;
}
```

```cpp
/*dfs版本*/
#include<iostream>
using namespace std;
int a[6][6];//用来储存地图和标记作用
int f1[4] = {0,0,-1,1};
int f2[4] = {1,-1,0,0};//方向数组

int sx, sy, fx, fy;//代表了起点和终点的坐标
int m, n, t;//m代表行，n代表列，t代表障碍数
int ans = 0;//总的条数

void dfs(int sx, int sy)
{
    int nx, ny;
    if(sx == fx && sy == fy){//终点的判定条件
        ans++;
        return;
    }

    a[sx][sy] = 1; //走过的要进行标记

    for(int i = 0; i < 4; ++i){//枚举上下左右四个方向
        nx = sx + f1[i];
        ny = sy + f2[i];
        if(nx < 1 || nx > m || ny < 1 || ny > n)//边界的判定条件
            continue;
        if(a[nx][ny] == 0){
            a[nx][ny] = 1;
            dfs(nx,ny);
            a[nx][ny] = 0;
        }
    }
}

int main()
{
    cin >> m >> n >> t;
    cin >> sx >> sy >> fx >> fy;
    int x, y;
    for(int i = 0; i < t; ++i){
        cin >> x >> y;
        a[x][y] = 1;//标记障碍的位置
    }

    dfs(sx,sy);
    cout << ans << endl;
    return 0;
}
```

### 矩阵连乘
##### 分析最优解的结构：
用A[i,j]表示AiAi+1…Aj乘积的结果矩阵，在对AiAi+1…Aj进行加括号时就相当于在某个位置k（i≤k≤j）处将矩阵链划分成两个部分。首先计算AiAi+1…Ak，在计算Ak…Aj，再将二者的结果在进行乘积运算得到最终结果。计算代价有三部分构成：A[i,k]的计算代价、A[k,j]的计算代价、二者相乘的计算代价。其中，在第一次得出k的划分后，在计算A[i,k]时，可继续进行寻找最优解k‘的操作，后半部分亦是如此。最终用二者的最优解方案得出的结果相乘得到总的最优解。
##### 建立递归方案：用m\[i]\[j]表示最少乘次数
①i=j时，m\[i]\[j]=0；
②i<j时，m\[i]\[j]=m\[I,k]+m\[k+1,j]+pi-1pkpj，由于在计算是并不知道断开点k的位置，所以k还未定。不过k的位置只有j-i个可能。因此，k是这j-i个位置使计算量达到最小的那个位置。(此处的k是假定k已知，实际上我们是不知道的，需要通过穷举得到)
##### 有如下递推关系：
![[Pasted image 20231225133927.png]]
```cpp
#include<stdio.h>
#include <iostream>
using namespace std;
int p[100];       // 存储第一个矩阵的行数以及所有矩阵的列数
int m[100][100];  // 存储最优值
int s[100][100];  // 存储断开位置

void MatrixChain(int n, int p[], int m[][100], int s[][100])
{
    /*用m[i][j]表示a[i]~a[j]最少乘次数*/
    for (int i = 1; i <= n; i++) m[i][i] = 0;
  
    for (int r = 2; r <= n; r++) {            //r: Ai和Aj之间的长度，即A的个数
        for (int i = 1; i <= n - r + 1; i++) {//第i行
            int j = i + r - 1;
            m[i][j] = m[i + 1][j] + p[i - 1] * p[i] * p[j];//在i处取得最优解
            s[i][j] = i;//断开位置
            for (int k = i + 1; k < j; k++) {//如果有更小的则被替换
                int t = m[i][k] + m[k + 1][j] + p[i - 1] * p[k] * p[j];//从k处断开
                if (t < m[i][j])
                {
                    m[i][j] = t;
                    s[i][j] = k;//断点
                }
            }
        }
    }
}
void printConclusions(int s[100][100], int i, int j)//输出结果
{
    if (i == j)
        cout << "A" << i;
    else
    {
        cout << "(";
        printConclusions(s, i, s[i][j]);
        printConclusions(s, s[i][j] + 1, j);
        cout << ")";
    }

}

int main()
{
    cout << "请输入矩阵的个数" << endl;
    int n;
    cin >> n;

    cout << "请依次输入第一个矩阵的行数以及所有矩阵的列数" << endl;
    for (int i = 0; i <= n; i++) {
        cin >> p[i];
    }

    MatrixChain(n, p, m, s);
    cout << "最少的相乘次数是:" << m[1][n] << endl;

    cout << "结果为:" << endl;
    printConclusions(s, 1, n);

    return 0;
}
```

### 活动安排问题
```cpp
#include<iostream>
#include<algorithm>
#define maxn 1000009
using namespace std;
int n;
struct node{
   int st;
   int ft;
}a[maxn];

bool cmp(node hbx1,node hbx2)
{
    return hbx1.ft < hbx2.ft;
}

int main()
{
    cin >> n;
    for(int i=1;i<=n;i++) cin >> a[i].st >> a[i].ft;
    sort(a+1,a+n+1,cmp);

    int wall = 0;
    int ans = 0;
    for(int i=1;i<=n;i++)
    {
        if(a[i].st < wall) continue;
        else {
            wall = a[i].ft;
            ans++;
        }
    }
    cout << ans <<endl;
    return 0;
}

/*
#include<bits/stdc++.h>
using namespace std;
int n;
struct node{
    int a;//开始时间
    int b;//结束时间
}tme[1000009];

bool cmp(node A,node B)//设置比较的回调函数
{
    return A.b < B.b;
}

int main()
{
    cin >> n;
    for(int i=1;i<=n;i++){
        cin >> tme[i].a >> tme[i].b;
    }
    sort(tme+1,tme+1+n,cmp);
    queue<node> que;
    while(!que.empty()) que.pop();
    que.push(tme[1]);
    for(int i=1;i<=n;i++)
    {
        if(tme[i].a >= que.back().b)
        {
            que.push(tme[i]);
        }
    }
    cout<<que.size()<<endl;
    return 0;
}
*/
```
### 装载问题
假设有两只船，各可以载重c1，c2。同时有n个集装箱，每个集装箱重量为w[i]，问能否将集装箱放入船内，能放入则就最优载重。

其实和背包问题如出一辙。如果一个给定的装载问题有解，则我们采用的策略应该是：先将第一艘轮船尽可能装满，然后将剩余的集装箱上第二艘轮船（如果不能把所有物品装入第二艘船那么问题无解）。

将第一艘轮船尽可能装满等价于选取全体集装箱的一个子集，使该子集中集装箱重量之和最接近c1（即求出最解）。由此可知，装载问题等价于特殊的0-1背包问题。

递归算法解决：没有到达叶结点，先考虑是否满足约束条件，若满足就加上该物品（进入左子树），将解加到解数组里面，更新cw即cw+=w[t]，再判断下一步(t+1)是否能左走，能则一直向左走，直到遇到叶结点或者cw+w[i]>c1则不能向左走。

遇到叶子结点则更新bestw，bestx、不能向左走就只能判断能向右走。然后就要考虑是否需要进入右子树，那就要判断当前的结点是否满足上界函数即cw+r>bestw：满足则进入右子树，回溯到右子树必须得先回到父节点，那么之前在左边加上的质量就要减掉即cw-w[t]，然后才可以进入右子树即执行（t+1）步。不满足的话即便把所有的剩余质量都加到船上也不能达到更大的bestw，所以直接舍弃剩余右子树，不再进行搜索。

```cpp
#include<iostream>
#define NUM 100
using namespace std;
int n;     //集装箱数量
int c1,c2; //轮船载重量
int cw;    //当前轮船载重量
int r;     //剩余集装箱重量
int w[100],bestw,x[100],bestx[100];  //最优载重,最优解

//t从1开始
void BackTrack(int t)
{
    //到达叶子节点
    if(t>n)
    {
        cout << cw << endl;
        if(cw > bestw) 
        {
            for(int i = 1; i <= n; i++)
            {
                if(x[i])
                    bestx[i] = i;
                else
                    bestx[i]=x[i];
            }
            
            bestw = cw;
            return;
        }
    }
    else {
        r -= w[t]; //更新剩余集装箱重量
        if(cw + w[t] <= c1) { //没有超出载重量，判断是否可以向左
            x[t] = 1;
            cw += w[t];
            cout << "左:" << t << " " << bestw << endl;
            BackTrack(t+1);
            cw -= w[t]; //cw要还原原先的载重，因为还要判断其他路线才能找到最优解
        }
        
        if(cw + r > bestw) { //判断是否可以向右
            cout << "右:" << t << " " << bestw << endl;
            x[t] = 0;
            BackTrack(t+1);
        }
        r += w[t]; //还原剩余集装箱重量走其他路线
    }
}

int main(){
    //读入数据；
    cout << "请输入AB船的容量c1、c2:";
    cin  >> c1 >> c2;
    cout << "输入物品的数量:";
    cin >> n;
    cout << "依次输入物品的质量:";
    for(int i = 1; i <= n; i++){//初始化，bestw=0
        cin >> w[i];
        r += w[i];
    }
    //递归回溯
    BackTrack(1);
    if(r-bestw > c2)//无解 
        cout << "没有装载方案" << endl;
    else {//有解
        cout << "最有载重为:" << bestw << endl;
        cout << "分别为第" ;
        for(int i = 1; i <= n; i++)
            if(bestx[i]) cout << bestx[i] << " ";
        cout << "个箱子" << endl;
    }
    return 0;
}
```
### 数字三角形
```cpp
#include <iostream>
#include <algorithm>
#include <iomanip>
#define maxn 555
#define INF 100000009
using namespace std;
/*
dp题型中一般下标从1开始，因为存在f[i-1][...]这类坑
*/
int n;
int a[maxn][maxn];
int f[maxn][maxn];

int main()
{
    cin >> n;
    for(int i=1;i<=n;i++)
    {
        for(int j=1;j<=i;j++)
        {
            cin >> a[i][j];
        }
    }

    for(int i=0;i<=n;i++)        // 注意这里必须从0开始
    {
        for(int j=0;j<=i+1;j++)  // 注意这里必须从0开始，到i+1结束（要完整包围）
        {
            f[i][j] = -INF;
        }
    }
    
    f[1][1] = a[1][1];

    for(int i=2;i<=n;i++)
    {
        for(int j=1;j<=i;j++)
        {
            f[i][j] = max(f[i-1][j-1]+a[i][j] , f[i-1][j]+a[i][j]);
        }
    }

    int ans = -INF;
    for(int i=1;i<=n;i++) ans = max(ans,f[n][i]);
    cout << ans <<endl;
    return 0;
}
```
### 搜索
##### 树与图的存储
```cpp
树是一种特殊的图，与图的存储方式相同。
对于无向图中的边ab，存储两条有向边a->b, b->a。
因此我们可以只考虑有向图的存储。

(1) 邻接矩阵：g[a][b] 存储边a->b

(2) 邻接表：

// 对于每个点k，开一个单链表，存储k所有可以走到的点。h[k]存储这个单链表的头结点
int h[N], e[N], ne[N], idx;

// 添加一条边a->b
void add(int a, int b)
{
    e[idx] = b, ne[idx] = h[a], h[a] = idx ++ ;
}

// 初始化
idx = 0;
memset(h, -1, sizeof h);
```
##### 树与图的遍历
```cpp
时间复杂度 O(n+m), n 表示点数，m 表示边数

(1) 深度优先遍历 —— 模板题 AcWing 846. 树的重心

int dfs(int u)
{
    st[u] = true; // st[u] 表示点u已经被遍历过

    for (int i = h[u]; i != -1; i = ne[i])
    {
        int j = e[i];
        if (!st[j]) dfs(j);
    }
}

(2) 宽度优先遍历 —— 模板题 AcWing 847. 图中点的层次

queue<int> q;
st[1] = true; // 表示1号点已经被遍历过
q.push(1);

while (q.size())
{
    int t = q.front();
    q.pop();

    for (int i = h[t]; i != -1; i = ne[i])
    {
        int j = e[i];
        if (!st[j])
        {
            st[j] = true; // 表示点j已经被遍历过
            q.push(j);
        }
    }
}
```
##### 拓扑排序
```cpp
拓扑排序 —— 模板题 AcWing 848. 有向图的拓扑序列

时间复杂度 O(n+m), n 表示点数，m 表示边数

bool topsort()
{
    int hh = 0, tt = -1;

    // d[i] 存储点i的入度
    for (int i = 1; i <= n; i ++ )
        if (!d[i])
            q[ ++ tt] = i;

    while (hh <= tt)
    {
        int t = q[hh ++ ];

        for (int i = h[t]; i != -1; i = ne[i])
        {
            int j = e[i];
            if (-- d[j] == 0)
                q[ ++ tt] = j;
        }
    }

    // 如果所有点都入队了，说明存在拓扑序列；否则不存在拓扑序列。
    return tt == n - 1;
}
```

![[Pasted image 20231227213512.png]]
##### dijkstra算法
```cpp
朴素dijkstra算法 —— 模板题 AcWing 849. Dijkstra求最短路 I

时间复杂是 O(n2+m), n 表示点数，m 表示边数

int g[N][N];  // 存储每条边
int dist[N];  // 存储1号点到每个点的最短距离
bool st[N];   // 存储每个点的最短路是否已经确定

// 求1号点到n号点的最短路，如果不存在则返回-1
int dijkstra()
{
    memset(dist, 0x3f, sizeof dist);
    dist[1] = 0;

    for (int i = 0; i < n - 1; i ++ )
    {
        int t = -1;     // 在还未确定最短路的点中，寻找距离最小的点
        for (int j = 1; j <= n; j ++ )
            if (!st[j] && (t == -1 || dist[t] > dist[j]))
                t = j;

        // 用t更新其他点的距离
        for (int j = 1; j <= n; j ++ )
            dist[j] = min(dist[j], dist[t] + g[t][j]);

        st[t] = true;
    }

    if (dist[n] == 0x3f3f3f3f) return -1;
    return dist[n];
}

堆优化版dijkstra —— 模板题 AcWing 850. Dijkstra求最短路 II

时间复杂度 O(mlogn), n 表示点数，m 表示边数

typedef pair<int, int> PII;

int n;      // 点的数量
int h[N], w[N], e[N], ne[N], idx;       // 邻接表存储所有边
int dist[N];        // 存储所有点到1号点的距离
bool st[N];     // 存储每个点的最短距离是否已确定

// 求1号点到n号点的最短距离，如果不存在，则返回-1
int dijkstra()
{
    memset(dist, 0x3f, sizeof dist);
    dist[1] = 0;
    priority_queue<PII, vector<PII>, greater<PII>> heap;
    heap.push({0, 1});      // first存储距离，second存储节点编号

    while (heap.size())
    {
        auto t = heap.top();
        heap.pop();

        int ver = t.second, distance = t.first;

        if (st[ver]) continue;
        st[ver] = true;

        for (int i = h[ver]; i != -1; i = ne[i])
        {
            int j = e[i];
            if (dist[j] > distance + w[i])
            {
                dist[j] = distance + w[i];
                heap.push({dist[j], j});
            }
        }
    }

    if (dist[n] == 0x3f3f3f3f) return -1;
    return dist[n];
}
```
##### Bellman-Ford算法
```cpp
Bellman-Ford算法 —— 模板题 AcWing 853. 有边数限制的最短路

时间复杂度 O(nm), n 表示点数，m 表示边数

注意在模板题中需要对下面的模板稍作修改，加上备份数组，详情见模板题。

int n, m;       // n表示点数，m表示边数
int dist[N];        // dist[x]存储1到x的最短路距离

struct Edge     // 边，a表示出点，b表示入点，w表示边的权重
{
    int a, b, w;
}edges[M];

// 求1到n的最短路距离，如果无法从1走到n，则返回-1。
int bellman_ford()
{
    memset(dist, 0x3f, sizeof dist);
    dist[1] = 0;

    // 如果第n次迭代仍然会松弛三角不等式，就说明存在一条长度是n+1的最短路径，由抽屉原理，路径中至少存在两个相同的点，说明图中存在负权回路。
    for (int i = 0; i < n; i ++ )
    {
        for (int j = 0; j < m; j ++ )
        {
            int a = edges[j].a, b = edges[j].b, w = edges[j].w;
            if (dist[b] > dist[a] + w)
                dist[b] = dist[a] + w;
        }
    }

    if (dist[n] > 0x3f3f3f3f / 2) return -1;
    return dist[n];
}
```
##### spfa 算法
```cpp
spfa 算法（队列优化的Bellman-Ford算法） —— 模板题 AcWing 851. spfa求最短路

时间复杂度 平均情况下 O(m)，最坏情况下 O(nm), n 表示点数，m 表示边数

int n;      // 总点数
int h[N], w[N], e[N], ne[N], idx;       // 邻接表存储所有边
int dist[N];        // 存储每个点到1号点的最短距离
bool st[N];     // 存储每个点是否在队列中

// 求1号点到n号点的最短路距离，如果从1号点无法走到n号点则返回-1
int spfa()
{
    memset(dist, 0x3f, sizeof dist);
    dist[1] = 0;

    queue<int> q;
    q.push(1);
    st[1] = true;

    while (q.size())
    {
        auto t = q.front();
        q.pop();

        st[t] = false;

        for (int i = h[t]; i != -1; i = ne[i])
        {
            int j = e[i];
            if (dist[j] > dist[t] + w[i])
            {
                dist[j] = dist[t] + w[i];
                if (!st[j])     // 如果队列中已存在j，则不需要将j重复插入
                {
                    q.push(j);
                    st[j] = true;
                }
            }
        }
    }

    if (dist[n] == 0x3f3f3f3f) return -1;
    return dist[n];
}

spfa判断图中是否存在负环 —— 模板题 AcWing 852. spfa判断负环

时间复杂度是 O(nm), n 表示点数，m 表示边数

int n;      // 总点数
int h[N], w[N], e[N], ne[N], idx;       // 邻接表存储所有边
int dist[N], cnt[N];        // dist[x]存储1号点到x的最短距离，cnt[x]存储1到x的最短路中经过的点数
bool st[N];     // 存储每个点是否在队列中

// 如果存在负环，则返回true，否则返回false。
bool spfa()
{
    // 不需要初始化dist数组
    // 原理：如果某条最短路径上有n个点（除了自己），那么加上自己之后一共有n+1个点，由抽屉原理一定有两个点相同，所以存在环。

    queue<int> q;
    for (int i = 1; i <= n; i ++ )
    {
        q.push(i);
        st[i] = true;
    }

    while (q.size())
    {
        auto t = q.front();
        q.pop();

        st[t] = false;

        for (int i = h[t]; i != -1; i = ne[i])
        {
            int j = e[i];
            if (dist[j] > dist[t] + w[i])
            {
                dist[j] = dist[t] + w[i];
                cnt[j] = cnt[t] + 1;
                if (cnt[j] >= n) return true;       // 如果从1号点到x的最短路中包含至少n个点（不包括自己），则说明存在环
                if (!st[j])
                {
                    q.push(j);
                    st[j] = true;
                }
            }
        }
    }

    return false;
}
```
##### floyd算法
```cpp
floyd算法 —— 模板题 AcWing 854. Floyd求最短路

时间复杂度是 O(n3), n 表示点数

初始化：
    for (int i = 1; i <= n; i ++ )
        for (int j = 1; j <= n; j ++ )
            if (i == j) d[i][j] = 0;
            else d[i][j] = INF;

// 算法结束后，d[a][b]表示a到b的最短距离
void floyd()
{
    for (int k = 1; k <= n; k ++ )
        for (int i = 1; i <= n; i ++ )
            for (int j = 1; j <= n; j ++ )
                d[i][j] = min(d[i][j], d[i][k] + d[k][j]);
}
```
##### prim算法
```cpp
朴素版prim算法 —— 模板题 AcWing 858. Prim算法求最小生成树

时间复杂度是 O(n2+m), n 表示点数，m 表示边数

int n;      // n表示点数
int g[N][N];        // 邻接矩阵，存储所有边
int dist[N];        // 存储其他点到当前最小生成树的距离
bool st[N];     // 存储每个点是否已经在生成树中


// 如果图不连通，则返回INF(值是0x3f3f3f3f), 否则返回最小生成树的树边权重之和
int prim()
{
    memset(dist, 0x3f, sizeof dist);

    int res = 0;
    for (int i = 0; i < n; i ++ )
    {
        int t = -1;
        for (int j = 1; j <= n; j ++ )
            if (!st[j] && (t == -1 || dist[t] > dist[j]))
                t = j;

        if (i && dist[t] == INF) return INF;

        if (i) res += dist[t];
        st[t] = true;

        for (int j = 1; j <= n; j ++ ) dist[j] = min(dist[j], g[t][j]);
    }

    return res;
}
```
##### Kruskal算法
```cpp
Kruskal算法 —— 模板题 AcWing 859. Kruskal算法求最小生成树

时间复杂度是 O(mlogm), n 表示点数，m 表示边数

int n, m;       // n是点数，m是边数
int p[N];       // 并查集的父节点数组

struct Edge     // 存储边
{
    int a, b, w;

    bool operator< (const Edge &W)const
    {
        return w < W.w;
    }
}edges[M];

int find(int x)     // 并查集核心操作
{
    if (p[x] != x) p[x] = find(p[x]);
    return p[x];
}

int kruskal()
{
    sort(edges, edges + m);

    for (int i = 1; i <= n; i ++ ) p[i] = i;    // 初始化并查集

    int res = 0, cnt = 0;
    for (int i = 0; i < m; i ++ )
    {
        int a = edges[i].a, b = edges[i].b, w = edges[i].w;

        a = find(a), b = find(b);
        if (a != b)     // 如果两个连通块不连通，则将这两个连通块合并
        {
            p[a] = b;
            res += w;
            cnt ++ ;
        }
    }

    if (cnt < n - 1) return INF;
    return res;
}
```
##### 染色法判别二分图
```cpp
染色法判别二分图 —— 模板题 AcWing 860. 染色法判定二分图

时间复杂度是 O(n+m), n 表示点数，m 表示边数

int n;      // n表示点数
int h[N], e[M], ne[M], idx;     // 邻接表存储图
int color[N];       // 表示每个点的颜色，-1表示未染色，0表示白色，1表示黑色

// 参数：u表示当前节点，c表示当前点的颜色
bool dfs(int u, int c)
{
    color[u] = c;
    for (int i = h[u]; i != -1; i = ne[i])
    {
        int j = e[i];
        if (color[j] == -1)
        {
            if (!dfs(j, !c)) return false;
        }
        else if (color[j] == c) return false;
    }

    return true;
}

bool check()
{
    memset(color, -1, sizeof color);
    bool flag = true;
    for (int i = 1; i <= n; i ++ )
        if (color[i] == -1)
            if (!dfs(i, 0))
            {
                flag = false;
                break;
            }
    return flag;
}
```
##### 匈牙利算法
```cpp
匈牙利算法 —— 模板题 AcWing 861. 二分图的最大匹配

时间复杂度是 O(nm), n 表示点数，m 表示边数

int n1, n2;     // n1表示第一个集合中的点数，n2表示第二个集合中的点数
int h[N], e[M], ne[M], idx;     // 邻接表存储所有边，匈牙利算法中只会用到从第一个集合指向第二个集合的边，所以这里只用存一个方向的边
int match[N];       // 存储第二个集合中的每个点当前匹配的第一个集合中的点是哪个
bool st[N];     // 表示第二个集合中的每个点是否已经被遍历过

bool find(int x)
{
    for (int i = h[x]; i != -1; i = ne[i])
    {
        int j = e[i];
        if (!st[j])
        {
            st[j] = true;
            if (match[j] == 0 || find(match[j]))
            {
                match[j] = x;
                return true;
            }
        }
    }

    return false;
}

// 求最大匹配数，依次枚举第一个集合中的每个点能否匹配第二个集合中的点
int res = 0;
for (int i = 1; i <= n1; i ++ )
{
    memset(st, false, sizeof st);
    if (find(i)) res ++ ;
}
```
