# 基础算法

## 排序

### **快速排序**

```c++
void quick_sort(int q[], int l, int r)
{
    if (l >= r) return;

    int i = l - 1, j = r + 1, x = q[l + r >> 1];
    while (i < j)
    {
        do i ++ ; while (q[i] < x);
        do j -- ; while (q[j] > x);
        if (i < j) swap(q[i], q[j]);
    }
    quick_sort(q, l, j), quick_sort(q, j + 1, r);

}
```

### **归并排序**

```c++
void merge_sort(int q[], int l, int r)
{
    if (l >= r) return;

    int mid = l + r >> 1;
    merge_sort(q, l, mid);
    merge_sort(q, mid + 1, r);
    
    int k = 0, i = l, j = mid + 1;
    while (i <= mid && j <= r)
        if (q[i] <= q[j]) tmp[k ++ ] = q[i ++ ];
        else tmp[k ++ ] = q[j ++ ];
    
    while (i <= mid) tmp[k ++ ] = q[i ++ ];
    while (j <= r) tmp[k ++ ] = q[j ++ ];
    
    for (i = l, j = 0; i <= r; i ++, j ++ ) q[i] = tmp[j];

}
```

## **前缀和**

### 1.一维

```c++
for (int i = 1; i <= n; i++)
	cin >> a[i];
s[1] = a[1];
for (int i = 2;i <= n; i++)
	s[i] = s[i - 1] + a[i];
while (m--)
{
	int l, r;
	cin >> l >> r;
	printf("%d", s[r] - s[l-1]);
}
```

### 2.二维

```c++
int main()
{
	scanf("%d%d%d", &n, &m, &q);
	for (int i = 1; i <= n; i++)
		for (int j = 1; j <= m; j++)
			scanf("%d", &a[i][j]);

	for (int i = 1; i <= n; i++)
		for (int j = 1; j <= m; j++)
			s[i][j] = s[i - 1][j] + s[i][j - 1] - s[i - 1][j - 1] + a[i][j];

	while (q--)
	{
		int x1, y1, x2, y2;
		scanf("%d%d%d%d", &x1, &y1, &x2, &y2);
		printf("%d\n", s[x2][y2] - s[x2][y1 - 1] - s[x1 - 1][y2] + s[x1 - 1][y1 - 1]);
	}
	return 0;
}
```

## **差分**

### 1.一维

```c++
//m个操作，l，r，c表示[l,r]之间每个数加c
#include<iostream>
using namespace std;
const int N=100010;
int n, m;
int a[N], b[N];
int l, r, c;
void insert(int l,int r,int c)
{
	b[l] += c;
	b[r + 1] -= c;
}
int main()
{
	cin >> n >> m;
	for (int i = 1; i <= n; i++)
		cin >> a[i];
	while (m--)
	{
		cin >> l >> r >> c;
		insert(l, r, c);
	}
	for (int i = 2; i <= n; i++)
		b[i] += b[i - 1];
	for (int i = 1; i <= n; i++)
		cout<<a[i]+b[i]<<' ';
}
```

### 2二维

```c++
#include<iostream>
using namespace std;
const int N = 1010;
int n,m,q;
int a[N][N], b[N][N];
void insert(int x1, int y1, int x2, int y2, int c)
{
	b[x1][y1] += c;
	b[x2 + 1][y1] -= c;
	b[x1][y2 + 1] -= c;
	b[x2 + 1][y2 + 1] += c;
}
int main()
{
	cin >> n >> m >> q;
	for (int i = 1; i <= n; i++)
		for (int j = 1; j <= m; j++)
			cin >> a[i][j];
	while (q--)
	{
		int x1, y1, x2, y2, c;
		cin >> x1 >> y1 >> x2 >> y2 >> c;
		insert(x1, y1, x2, y2, c);
	}
	for (int i = 1; i <= n; i++)
		for (int j = 1; j <= m; j++)
			b[i][j] += b[i - 1][j] + b[i][j - 1] - b[i - 1][j - 1];
	for (int i = 1; i <= n; i++)
	{
		for (int j = 1; j <= m; j++)
			printf("%d ", b[i][j]+a[i][j]);
		puts("");//换行
	}
}
```

## **二分**

```c++
bool check(int x) {/* ... */ } // 检查x是否满足某种性质
// 区间[l, r]被划分成[l, mid]和[mid + 1, r]时使用：
int bsearch_1(int l, int r)
{
    while (l < r)
    {
        int mid = l + r >> 1;
        if (check(mid)) r = mid;    // check()判断mid是否满足性质
        else l = mid + 1;
    }
    return l;
}
// 区间[l, r]被划分成[l, mid - 1]和[mid, r]时使用：
int bsearch_2(int l, int r)
{
    while (l < r)
    {
        int mid = l + r + 1 >> 1;
        if (check(mid)) l = mid;
        else r = mid - 1;
    }
    return l;
}
```

```c++
bool check(double x) {/* ... */} // 检查x是否满足某种性质

double bsearch_3(double l, double r)
{
    const double eps = 1e-6;   // eps 表示精度，取决于题目对精度的要求
    while (r - l > eps)
    {
        double mid = (l + r) / 2;
        if (check(mid)) r = mid;
        else l = mid;
    }
    return l;
}
```

## **位运算**

```c++
int main()n的二进制的第k位是几
{
	int n = 10;
	for (int k = 3; k >= 0; k--)
    cout << (n >> k & 1) << endl;
}
```

```c++
int lowbit(int x)
{
	return x & -x;
}
int main()
{
	int n; cin >> n;
	while (n--)
	{
		int x; cin >> x;
		int res = 0;
		while (x) x -= lowbit(x), res++;
		cout << res << ' ';
	}
}
```

## **区间合并**

```c++
#include<iostream>
#include<algorithm>
#include<vector>
using namespace std;

typedef pair<int, int> PII;
int n;
vector<PII> segs;

void merge(vector<PII>& segs)
{
	vector<PII>  res;
	sort(segs.begin(), segs.end());
	
	int st = -2e9, ed = -2e9;
	for (auto seg:segs)
	{
		if (ed < seg.first)
		{
			if (st != -2e9) res.push_back({ st,ed });
			st = seg.first, ed = seg.second;
		}
		else ed = max(ed, seg.second);
	}
	if (st != -2e9) res.push_back({ st,ed });
	segs = res;
}
int main()
{
	cin >> n;
	for (int i = 0; i < n; i++)
	{
		int l, r;
		cin >> l >> r;
		segs.push_back({ l , r });
	}
	merge(segs);
	cout << segs.size() << endl;
	return 0;
}
```

## **离散化**

```c++
#include<iostream>
#include<vector>
#include <algorithm>
using namespace std;
vector<int> alls; // 存储所有待离散化的值
sort(alls.begin(), alls.end()); // 将所有值排序
alls.erase(unique(alls.begin(), alls.end()), alls.end());   // 去掉重复元素

// 二分求出x对应的离散化的值
int find(int x) // 找到第一个大于等于x的位置
{
    int l = 0, r = alls.size() - 1;
    while (l < r)
    {
        int mid = l + r >> 1;
        if (alls[mid] >= x) r = mid;
        else l = mid + 1;
    }
    return r + 1; // 映射到1, 2, ...n
}
```

# **数据结构**

## 链表

### 1.单链表

```c++
#include<iostream>
using namespace std;

const int N = 100010;
//头结点下标，节点值，节点的指针，储存当前已经用到的节点
int head, e[N], ne[N], idx;
//初始化
void init()
{
	head = -1;
	idx = 0;
}
//将x插到头结点
void add_to_head(int x)
{
	e[idx] = x;
	ne[idx] = head;
	head = idx;
	idx++;
}
//将x插到下标是k的点后面
void add(int k, int x)
{
	e[idx] = x;
	ne[idx] = ne[k];
	ne[k] = idx;
	idx++; 
}
//将x下标是k的点后面的点删除
void remove(int k)
{
	ne[k] = ne[ne[k]];
}
//遍历
int main()
{
	for (int i = head; i != -1; i = ne[i]);
}
```

### 2.双链表

```c++
#include<iostream>
using namespace std;

const int N = 100010;
int m;
int e[N], l[N], r[N],idx;
//初始化
void init()
{
	r[0] = 1;
	l[1] = 0;
	idx = 2;
}
//在下标为k的点的右边插入x( 若左边插add(l[k],x) )
void add(int k, int x)
{
	e[idx] = x;
	l[idx] = k;
	r[idx] = r[k];
	l[r[k]] = idx;
	r[k] = idx++;
}
//删除下标为k的点
void remove(int k, int x)
{
	r[l[k]] = r[k];
	l[r[k]] = l[k];
}
```

## 栈和队列

### 1.模拟栈和队列

```c++
#include<iostream>
using namespace std;

const int N=100010;	int x;
//*******************************栈
void zhan()
{
	int stk[N], tt = 0;// tt表示栈顶
	stk[++tt] = x;	// 向栈顶插入一个数
	tt--;	// 从栈顶弹出一个数
	stk[tt];	// 栈顶的值
	if (tt > 0)// 判断栈是否为空，如果 tt > 0，则表示不为空
	{

	}
}
//*******************************队列
void dui()
{
	int q[N], hh = 0, tt = -1;// hh 表示队头，tt表示队尾
	q[++tt] = x;// 向队尾插入一个数
	hh++;	// 从队头弹出一个数
	q[hh];// 队头的值
	if (hh <= tt)// 判断队列是否为空，如果 hh <= tt，则表示不为空
	{

	}
}
```

### 2.单调栈

```c++
//长度为N的整数序列，输出每个数左边第一个比它小的数，没有输出-1
#include<iostream>
using namespace std;

const int N = 1000;
int stk[N], tt, n;
int main()
{
	cin >> n;
	for (int i = 0; i < n; i++)
	{
		int x;
		cin >> x;
		while (tt && stk[tt] >= x) tt--;
		if (tt) cout << stk[tt] << ' ';
		else cout << -1 << ' ';
		stk[++tt] = x;
	}
	return 0;
}
```

### 3.单调队列

```c++
//滑动区间大小看k，找区间最大最小值
#define _CRT_SECURE_NO_WARNINGS
#include<iostream>
using namespace std;

const int N = 1e6 + 10;
int q[N], hh, tt;
int a[N], k;

int main()
{
	int n, k;
	cin >> n >> k;
	for (int i = 0; i < n; ++i) cin >> a[i];

	hh = 0, tt = -1;
	for (int i = 0; i < n; ++i)
	{
		if (hh <= tt && q[hh] < i + 1 - k) hh++;
		while (hh <= tt && a[q[tt]] >= a[i]) tt--; // 注意是从队尾往前去除较大数
		q[++tt] = i;
		if (i + 1 - k >= 0) cout << a[q[hh]] << ' ';
	}
	cout << endl;

	hh = 0, tt = -1;
	for (int i = 0; i < n; ++i)
	{
		if (hh <= tt && q[hh] < i + 1 - k) hh++;
		while (hh <= tt && a[q[tt]] <= a[i]) tt--; // 注意是从队尾往前去除较小数
		q[++tt] = i;
		if (i + 1 - k >= 0) cout << a[q[hh]] << ' ';
	}
}
```

## kmp

```c++
//字符串s为主串，p为模式串，p在s中包含多次，求出p在s中所有出现位置下标
//输入先p后s
#include<iostream>
using namespace std;

const int N = 10010,M=100010;
int n,m;
char p[N], s[M];
int ne[N];

int main()
{
	cin >> n >> p >> m >> s;
	int j = 0; ne[0] = 0;
	for (int i = 1; i < n; i++)//next
	{
		while (j > 0 && p[i] != p[j]) j = ne[j - 1];
		if (p[i] == p[j]) j++;
		ne[i] = j;
	}
	j = 0;
	for (int i = 0; i < m; i++)
	{
		while (j > 0 && s[i] != p[j]) j = ne[j - 1];
		if (s[i] == p[j]) j++;
		if(j == n)
		{
			printf("%d ", i - n+1);
			j = ne[j - 1];
		}
	}
}
```

## trie（字典树，前缀树）

```c++
//使用trie插入字符串，查找字符串
#include<iostream>
using namespace std;

const int N = 100010;
int son[N][26], cnt[N], idx;
char str[N];

void insert(char str[])//插入
{
	int p = 0;
	for (int i = 0; str[i]; i++)
	{
		int u = str[i] - 'a';
		if (!son[p][u]) son[p][u] = ++idx;
		p = son[p][u];
	}
	cnt[p]++;
}

int query(char str[])
{
	int p = 0;
	for (int i = 0; str[i]; i++)
	{
		int u = str[i] - 'a';
		if (!son[p][u]) return 0;
		p = son[p][u];
	}
	return cnt[p];
}
```

## 并查集

```c++
#include<iostream>
using namespace std;
// //1.将两个集合合并
//2.询问两个元素是否在一个集合当中
//3.size为集合数的个数
const int N = 100010;
int p[N]; //存储每个点的祖宗节点
int size_[N];//p[]存储每个点的祖宗节点, size[]只有祖宗节点的有意义，表示祖宗节点所在集合中的点的数量

int find(int x)// 返回x的祖宗节点+路径压缩
{
    if (p[x] != x) p[x] = find(p[x]);
    return p[x];
}

int main()
{
    int n; cin >> n;
    for (int i = 1; i <= n; i++)
    {
        p[i] = i;// 初始化，假定节点编号是1~n
        size_[i] = 1;
    }
    int a, b;
    a = find(a), b = find(b);
    if (a != b)
    {
        size_[b] += size_[a];// 合并a和b所在的两个集合
        p[a] = b;
    }
    
}
```

## 堆

```c++
/*I x，插入一个数 x；
PM，输出当前集合中的最小值；
DM，删除当前集合中的最小值（数据保证此时的最小值唯一）；
D k，删除第 k个插入的数；
C k x，修改第 k个插入的数，将其变为x；*/
#define _CRT_SECURE_NO_WARNINGS
#include<iostream>
using namespace std;

const int N = 1e5 + 10;

// h[N]存储堆中的值, h[1]是堆顶，x的左儿子是2x, 右儿子是2x + 1
// ph[k]存储第k个插入的点在堆中的位置
// hp[k]存储堆中下标是k的点是第几个插入的
int h[N], ph[N], hp[N], cnt;

void heap_swap(int a, int b)
{
	// 根据a和b的位置找到它们分别是第几个插入的元素，然后将其（在h数组中的）下标转换
	swap(ph[hp[a]], ph[hp[b]]);
	// 将两个位置存的是第几号元素转换
	swap(hp[a], hp[b]);
	// 最后再转换值(这三个语句位置可以换，但是从上到下逐渐变短的话比较美观)
	swap(h[a], h[b]);
}
void down(int x)
{
	int t = x;
	if (2 * x <= cnt && h[2 * x] < h[t]) t = 2 * x;
	if (2 * x + 1 <= cnt && h[2 * x + 1] < h[t]) t = 2 * x + 1;
	if (t != x)
	{
		heap_swap(x, t);
		down(t);
	}
}
void up(int x)
{
	while (x / 2 && h[x] < h[x / 2])
	{
		heap_swap(x, x / 2);
		x >>= 1;
	}
}

int main()
{
	cin.tie(0);
	int n, m = 0;
	cin >> n;
	while (n--)
	{
		string s;
		int k, x;
		cin >> s;
		if (s == "I")
		{  
			cin >> x;
			cnt++;
			m++;
			ph[m] = cnt, hp[cnt] = m;
			h[cnt] = x;
			up(cnt);
		}
		else if (s == "PM") cout << h[1] << endl;
		else if (s == "DM")
		{
			heap_swap(1, cnt);
			cnt--;
			down(1);
		}
		else if (s == "D")
		{
			cin >> k;
			k = ph[k];
			heap_swap(k, cnt);
			cnt--;
			up(k);
			down(k);
		}
		else if (s == "C")
		{
			cin >> k >> x;
			k = ph[k];
			h[k] = x;
			up(k);
			down(k);
		}
		else cout << "error" << endl;
	}
	return 0;
}

```

## C++ STL

```c++
vector, 变长数组，倍增的思想（分配空间时所需时间与空间大小无关，于申请次数有关）
size()  返回元素个数
empty()  返回是否为空，空为true，不空为false
clear()  清空
front() / back()  返回第一个数/最后一个数
push_back() / pop_back()  在最后插入一个数/删除最后一个数
begin() / end()  迭代器，第0个数/第n+1个数
[]随机寻址
遍历方式：
for(int i=0;i<a,size();i++)
for(auto i=a.begin();i!=a.end();i++)
for(auto x:a) 
支持比较运算，按字典序

pair<int, int>
first, 第一个元素
second, 第二个元素
支持比较运算，以first为第一关键字，以second为第二关键字（字典序）

string，字符串
size() / length()  返回字符串长度
empty()
clear()
substr(起始下标，(子串长度))  返回子串
c_str()  返回字符串所在字符数组的起始地址

queue, 队列
size()
empty()
push()  向队尾插入一个元素
front()  返回队头元素
back()  返回队尾元素
pop()  弹出队头元素
q=queue<int>();  清空clear

priority_queue, 优先队列，默认是大根堆
size()
empty()
push()  插入一个元素
top()  返回堆顶元素
pop()  弹出堆顶元素
定义成小根堆的方式：priority_queue<int, vector<int>, greater<int>> q;

stack, 栈
size()
empty()
push()  向栈顶插入一个元素
top()  返回栈顶元素
pop()  弹出栈顶元素

deque, 双端队列     
size()
empty()
clear()
front() / back()
push_back() / pop_back()
push_front() / pop_front()
begin() / end()
[]

set, map, multiset, multimap, 基于平衡二叉树（红黑树），动态维护有序序列
size()
empty()
clear()
begin() / end()
++, --返回前驱和后继，时间复杂度 O(logn)

set / multiset
insert()  插入一个数
find()  查找一个数
count()  返回某一个数的个数
erase()
	(1) 输入是一个数x，删除所有x   O(k + logn)
	(2) 输入一个迭代器，删除这个迭代器
lower_bound() / upper_bound()
lower_bound(x)  返回大于等于x的最小的数的迭代器
upper_bound(x)  返回大于x的最小的数的迭代器

map / multimap
insert()  插入的数是一个pair
erase()  输入的参数是pair或者迭代器
find()
[]  注意multimap不支持此操作。 时间复杂度是 O(logn)
lower_bound() / upper_bound()

unordered_set, unordered_map, unordered_multiset, unordered_multimap, 哈希表
和上面类似，增删改查的时间复杂度是 O(1)
不支持 lower_bound() / upper_bound()， 迭代器的++，--

bitset, 圧位
bitset<10000> s;
~, &, | , ^
>> , <<
==, !=
[]

count()  返回有多少个1

any()  判断是否至少有一个1
none()  判断是否全为0

set()  把所有位置成1
set(k, v)  将第k位变成v
reset()  把所有位变成0
flip()  等价于~
flip(k) 把第k位取反
```

 

## **Hash**

### 1.开放寻址法

```c++
#include<cstring>
#include<iostream>

using namespace std;
const int N = 200003,null=0x3f3f3f3f;//质数（2到3倍）
int h[N];

int find(int x)
{
	int k = (x % N + N) % N;
	while (h[k] != null && h[k] != x)
	{
		k++;
		if (k == N) k = 0;
	}
	return k;
}

int main()
{
	int n;
	scanf("%d", &n);
	memset(h,0x3f, sizeof h);//按字节初始化，int 是四个字节，一般为0，-1
	while (n--)
	{
		char op[2];
		int x;
		scanf("%s%d", op, &x);
		int k = find(x);
		if (*op == 'I') h[k] = x;
		else
		{
			if (h[k]!=null) puts("yes");
			else puts("No");
		}
	}
	return 0;
}
```

### 2.链表法

```c++
#include<cstring>
#include<iostream>

using namespace std;
const int N = 10003;//质数
int h[N], e[N], ne[N], idx;

void insert(int x)
{
	int k = (x % N + N) % N;
	e[idx] = x;
	ne[idx] = h[k];
	h[k] = idx++;
}

bool find(int x)
{
	int k = (x % N + N) % N;
	for (int i = h[k]; i != -1; i = ne[i])
		if (e[i] == x)
			return true;
	return false;
}

int main()
{
	int n;
	scanf("%d", &n);
	memset(h, -1, sizeof h);
	while (n--)
	{
		char op[2];
		int x;
		scanf("%s%d", op, &x);

		if (*op == 'I') insert(x);
		else
		{
			if (find(x)) puts("yes");
			else puts("No");
		}
	}
	return 0;
}
```

### 3.字符串

```c++
#include<iostream>
using namespace std;

typedef unsigned long long ULL;
const int N = 100010, P = 131;

int n, m;
char str[N];
ULL h[N], p[N];

ULL get(int l, int r)
{
	return h[r] - h[l - 1] * p[r - l + 1];
}

int main()
{
	scanf("%d%d%s", &n, &m, str + 1);
	p[0] = 1;
	for (int i = 1; i <= n; i++)
	{
		p[i] = p[i - 1] * P;
		h[i] = h[i - 1] * P + str[i];
	}
	while (m--)
	{
		int l1, r1, l2, r2;
		scanf("%d%d%d%d", &l1, &r1, &l2, &r2);
		if (get(l1, r1) == get(l2, r2))
			printf("Yes");
		else printf("No");
	}
	return 0;

}
```

## **二叉树**

```c++
#include <iostream>
#include <unordered_set>
using namespace std;

int tree[1110],ll[1110],idxx;
int s,t,m;

int getpower2(int n)
{
	int cnt = 0;
	while (n > 1)
	{
		n >>= 1;
		cnt++;
	}
	return cnt;
}

void creattree(int idx,int n)
{
	if (n > s) return;
	else
	{
		int a; cin >> a;
		tree[idx] = a;
		creattree(idx * 2, n + 1);
		creattree(idx * 2 + 1, n + 1);
	}
}

void puttree_hou(int idx, int n)
{
	if (n > s) return;
	else
	{
		puttree_hou(idx * 2, n + 1);
		puttree_hou(idx * 2 + 1, n + 1);
		cout << tree[idx]; m++;
		if(m<t)
		cout << ' ';
	}
}

void gettree_zhong(int idx, int n)
{
	if (n > s) return;
	else
	{
		gettree_zhong(idx * 2, n + 1);
		ll[idxx++] = tree[idx];
		gettree_zhong(idx * 2 + 1, n + 1);
		
	}
}

int main()
{
	cin >> t;
	s= getpower2(t + 1);
	creattree(1, 1);
	gettree_zhong(1, 1);

	bool st=false;
	if (ll[0] <= ll[t - 1])
	{
		for (int i = 1; i < t; i++)
		{
			if (ll[i] < ll[i - 1])
			{
				st = true;
				break;
			}
		}
	}
	else
	{
		for (int i = 1; i < t; i++)
		{
			if (ll[i] > ll[i - 1])
			{
				st = true;
				break;
			}
		}
	}
	if (!st)
	{
		cout << "YES" << endl;
		puttree_hou(1, 1);
	}
	else
		cout << "NO";
}
```

```c++
#include<iostream>
#include<algorithm>
#include<vector>
using namespace std;

vector<int> a, b;
int n;


void print(int l1, int r1, int l2, int r2) {
    if (l1 > r1 || l2 > r2)
        return;
    // 在后序遍历序列 a 中找到根节点
    int root = a[r2];
    // 在中序遍历序列 b 中找到根节点的位置
    auto it = find(b.begin() + l1, b.begin() + r1 + 1, root);
    int x = it - b.begin();
    // 计算左子树的节点数量
    int left_size = x - l1;


    cout << root << ' ';

    print(l1, x - 1, l2, l2 + left_size - 1);

    print(x + 1, r1, l2 + left_size, r2 - 1);
}

int main() {
    cin >> n;
    for (int i = 0; i < n; i++) //后
    {
        int m;
        cin >> m;
        a.push_back(m);
    }
    for (int i = 0; i < n; i++) //中
    {
        int m;
        cin >> m;
        b.push_back(m);
    }
    print(0, n - 1, 0, n - 1);
    return 0;
}
```

```c++
#include<iostream>
#include<algorithm>
#include<vector>
#include<cstring>
using namespace std;

vector<int> a, b;
int n;
int tree[90], midx;


void print(int l1, int r1, int l2, int r2, int idx) {
    if (l1 > r1 || l2 > r2)
        return;
    // 在后序遍历序列 a 中找到根节点
    int root = a[r2];
    // 在中序遍历序列 b 中找到根节点的位置
    auto it = find(b.begin() + l1, b.begin() + r1 + 1, root);
    int x = it - b.begin();
    // 计算左子树的节点数量
    int left_size = x - l1;


    tree[idx] = root;
    if (midx < idx)
        midx = idx;
    print(l1, x - 1, l2, l2 + left_size - 1, idx * 2);
    print(x + 1, r1, l2 + left_size, r2 - 1, idx * 2 + 1);

}

int main() {
    cin >> n;
    for (int i = 0; i < n; i++) {
        int m;
        cin >> m;
        a.push_back(m);
    }
    for (int i = 0; i < n; i++) {
        int m;
        cin >> m;
        b.push_back(m);
    }

    memset(tree, -1, sizeof tree);
    print(0, n - 1, 0, n - 1, 1);
    int f = midx;
    for (int i = 1; i <= f; i++)
    {
        if (tree[i] != -1)
        {
            if (i != 1)
                cout << ' ';
            cout << tree[i];
        }
    }
    return 0;
}
```

## **哈夫曼树**

```c++
#include<iostream>
#include<vector>
#include<queue>
using namespace std;

struct hafu {
    int w;  // 权值
    int p;  // 父节点索引
    int l;  // 左儿子索引
    int r;  // 右儿子索引
};

struct cmp {
    bool operator()(const pair<int, int>& a, const pair<int, int>& b) {
        return a.first > b.first;  // 小根堆（权重小的优先）
    }
};

// 函数声明
void maketree(const vector<int>& arr, vector<hafu>& node, int n, int m);
int WPL(const vector<hafu>& node, int n);

int main() {
    int n;
    cin >> n;
    int m = 2 * n - 1;  // 总节点数，应在n确定后计算

    vector<int> arr(n);
    for (int i = 0; i < n; i++) {
        cin >> arr[i];
    }

    vector<hafu> node(m);  // 初始化节点向量，指定大小为m
    maketree(arr, node, n, m);

    cout << WPL(node, n) << endl;

    return 0;
}

// 建立哈夫曼树
void maketree(const vector<int>& arr, vector<hafu>& node, int n, int m) {
    // 初始化叶子节点
    for (int i = 0; i < n; i++) {
        node[i] = { arr[i], -1, -1, -1 };
    }

    // 初始化中间节点
    for (int i = n; i < m; i++) {
        node[i] = { 0, -1, -1, -1 };
    }

    priority_queue<pair<int, int>, vector<pair<int, int>>, cmp> heap;
    for (int i = 0; i < n; i++) {
        heap.push({ node[i].w, i });  // 叶子结点入队
    }

    for (int i = n; i < m; i++) {
        auto ll = heap.top(); heap.pop();
        auto rr = heap.top(); heap.pop();

        // 存入和节点
        node[i].w = ll.first + rr.first;
        node[i].l = ll.second;
        node[i].r = rr.second;
        node[ll.second].p = i;
        node[rr.second].p = i;

        heap.push({ node[i].w, i });
    }
}

// 计算WPL
int WPL(const vector<hafu>& node, int n) {
    int wpl = 0;
    for (int i = 0; i < n; i++) {
        int d = 0;  // 深度
        int j = i;
        while (node[j].p != -1) {
            d++;
            j = node[j].p;  // 修正：应该访问当前节点j的父节点，而非i的
        }
        wpl += node[i].w * d;
    }
    return wpl;
}

```

## **树状数组**

### 一维

```c++
用法一(单点增加+范围查询)tree维护原始数组
#include<iostream>
using namespace std;

typedef long long LL;
const int N = 500010;
LL tree[N];
int n, m;

int lowbit(int i)
{
	return i & -i;
}

void add(int i,int a)
{
	while (i <= n)
	{
		tree[i] += a;
		i += lowbit(i);
	}
}

LL sum(int i)//1~i的sum
{
	LL ans = 0;
	while (i>0)
	{
		ans += tree[i];
		i -= lowbit(i);
	}
	return ans;
}

int main()
{

	cin >> n >> m;
	for (int i = 1; i <= n; i++)
	{
		int a;
		cin >> a;
		add(i, a);
	}

	for (int i = 0; i < m; i++)
	{
		int a, b, c;
		cin >> a >> b >> c;
		if (a == 1)
			add(b, c);
		if (a == 2)
		{
			LL sb = sum(b-1);
			LL sc = sum(c);
			cout << sc - sb << endl;
		}
	}
}

用法二(范围增加+单点查询)tree维护差分数组
//函数不变
int main()
{
	cin >> n >> m;
	for (int i = 1; i <= n; i++)
	{
		int a;
		cin >> a;
		add(i, a);
		add(i + 1, -a);
	}

	for (int i = 0; i < m; i++)
	{
		int a; cin >> a;
		if (a == 1)
		{
			int b, c, d;
			cin >> b >> c >> d;
			add(b, d);
			add(c + 1, -d);
		}
		else if (a == 2)
		{
			int b;
			cin >> b;
			cout << sum(b) << endl;
		}
	}
}

用法三(范围增加+范围查询)tree1维护差分D数组  tree2维护(i-1)*D数组
#include<iostream>
using namespace std;

typedef long long LL;
const int N = 500010;
int n, m;
LL tree1[N];
LL tree2[N];

int lowbit(int i)
{
	return i & -i;
}

void add_tree(LL tree[],int i, int a)
{
	while (i <= n)
	{
		tree[i] += a;
		i += lowbit(i);
	}
}

LL sum(LL tree[],int i)
{
	LL ans = 0;
	while (i > 0)
	{
		ans += tree[i];
		i -= lowbit(i);
	}
	return ans;
}
//原始数组i到j,每个数加a
void add(int i, int j, int a)
{
	add_tree(tree1, i, a);
	add_tree(tree1, j + 1, -a);
	add_tree(tree2, i, (i - 1) * a);
	add_tree(tree2, j + 1, -j * a);
}
//原始数组i到j范围的累加和
LL range(int i, int j)
{
	return j * sum(tree1, j) - sum(tree2, j) - ((i - 1) * sum(tree1, i - 1) - sum(tree2, i - 1));
}

int main()
{
	cin >> n >> m;
	for (int i = 1; i <= n; i++)
	{
		int a;
		cin >> a;
		add(i, i, a);
	}

	for (int i = 0; i < m; i++)
	{
		int a; cin >> a;
		if (a == 1)
		{
			int b, c, d;
			cin >> b >> c >> d;
			add(b, c, d);
		}
		else if (a == 2)
		{
			int b,c;
			cin >> b >> c;
			cout << range(b,c) << endl;
		}
	}
}
```

## **线段树**

```c++
#include<iostream>
#include<cmath>
#include<algorithm>
using namespace std;

typedef long long LL;
const int N = 100010;
LL sum[N*4], add[N*4],arr[N];

void lazy(int i, LL v, int n)
{
	sum[i] += v * n;
	add[i] += v;
}
//累加
void up(int i)
{
	sum[i] = sum[i << 1] + sum[i << 1 | 1];//父范围=左范围+右范围
}

//懒信息下发
void down(int i, int ln, int rn)
{
	if (add[i] != 0)
	{
		lazy(i << 1, add[i], ln);//左发
		lazy(i << 1 | 1, add[i], rn);//右发
		add[i] = 0;	//懒信息清空
	}
}

//建树
void build(int l, int r, int i)
{
	if (l == r)
		sum[i] = arr[l];
	else
	{
		int mid = (l + r) >> 1;
		build(l, mid, i << 1);
		build(mid + 1, r, i << 1 | 1);
		up(i);
	}
	add[i] = 0;
}

void addv(int jobl, int jobr, LL jobv, int l, int r, int i)
{
	if (jobl <= l && r <= jobr)
		lazy(i, jobv, r - l + 1);
	else
	{
		int mid = (l + r) >> 1;
		down(i, mid - l + 1, r - mid);
		if (jobl <= mid)
			addv(jobl, jobr, jobv, l, mid, i << 1);
		if (jobr > mid)
			addv(jobl, jobr, jobv, mid + 1, r, i << 1 | 1);

		up(i);
	}
}

LL query(int jobl, int jobr, int l, int r, int i)
{
	if (jobl <= l && r <= jobr)
		return sum[i];

	int mid = (l + r) >> 1;
	down(i, mid - l + 1, r - mid);

	LL ans = 0;
	if (jobl <= mid)
		ans += query(jobl, jobr, l, mid, i << 1);
	if (jobr > mid)
		ans += query(jobl, jobr, mid + 1, r, i << 1 | 1);
	return ans;
}

int main()
{
	int n, m;
	cin >> n >> m;

	for (int i = 1; i <= n; i++)
		cin >> arr[i];
	build(1, n, 1);

	for (int i = 0; i < m; i++)
	{
		int a; cin >> a;
		if (a == 1)
		{
			int x, y;
			LL k;
			cin >> x >> y >> k;
			addv(x, y, k, 1, n, 1);
		}
		else if (a == 2)
		{
			int x, y;
			cin >> x >> y;
			cout << query(x, y,1,n,1) << endl;
		}
	}
}
```

## 树形DP

```c++
#include<iostream>
#include<map>
#include<cmath>
#include<cstring>
using namespace std;//给一棵树（每个节点代表员工，有 “快乐值” 属性），选一些节点（员工），要求选的节点不能是直接上下级（父和子不能同时选），最终让选中节点的快乐值总和最大 。

const int N = 6010;
int n;
int happy[N];
int h[N], e[N], ne[N], idx;
int f[N][2];         
bool has_father[N];//找头节点

void add(int a, int b)
{
	e[idx] = b, ne[idx] = h[a], h[a] = idx++;
}

void dfs(int u)
{
	f[u][1] = happy[u];

	for (int i = h[u]; i != -1; i = ne[i])//遍历父节点的所有子节点
	{
		int j = e[i];
		dfs(j);

		f[u][0] += max(f[j][0], f[j][1]);//f[u][0] 表示不选择第 u 个员工参加舞会时，以 u 为根的子树能获得的最大快乐值
		f[u][1] += f[j][0];              //f[u][1] 表示选择第 u 个员工参加舞会时，以 u 为根的子树能获得的最大快乐值
	}
}

int main()
{
	cin >> n;
	for (int i = 1; i <= n; i++)
		cin >> happy[i];

	memset(h, -1, sizeof h);
	for (int i = 0; i < n - 1; i++)
	{
		int a, b;
		cin >> a >> b;
		has_father[a] = true;
		add(b, a);
	}

	int root = 1;
	while (has_father[root]) root++;
	dfs(root);
	cout << max(f[root][0], f[root][1]);
}
```

## **哈夫曼树**

```c++
#include<iostream>
#include<vector>
#include<queue>
using namespace std;

struct hafu {
    int w;  // 权值
    int p;  // 父节点索引
    int l;  // 左儿子索引
    int r;  // 右儿子索引
};

struct cmp {
    bool operator()(const pair<int, int>& a, const pair<int, int>& b) {
        return a.first > b.first;  // 小根堆（权重小的优先）
    }
};

// 函数声明
void maketree(const vector<int>& arr, vector<hafu>& node, int n, int m);
int WPL(const vector<hafu>& node, int n);

int main() {
    int n;
    cin >> n;
    int m = 2 * n - 1;  // 总节点数，应在n确定后计算

    vector<int> arr(n);
    for (int i = 0; i < n; i++) {
        cin >> arr[i];
    }

    vector<hafu> node(m);  // 初始化节点向量，指定大小为m
    maketree(arr, node, n, m);

    cout << WPL(node, n) << endl;

    return 0;
}

// 建立哈夫曼树
void maketree(const vector<int>& arr, vector<hafu>& node, int n, int m) {
    // 初始化叶子节点
    for (int i = 0; i < n; i++) {
        node[i] = { arr[i], -1, -1, -1 };
    }

    // 初始化中间节点
    for (int i = n; i < m; i++) {
        node[i] = { 0, -1, -1, -1 };
    }

    priority_queue<pair<int, int>, vector<pair<int, int>>, cmp> heap;
    for (int i = 0; i < n; i++) {
        heap.push({ node[i].w, i });  // 叶子结点入队
    }

    for (int i = n; i < m; i++) {
        auto ll = heap.top(); heap.pop();
        auto rr = heap.top(); heap.pop();

        // 存入和节点
        node[i].w = ll.first + rr.first;
        node[i].l = ll.second;
        node[i].r = rr.second;
        node[ll.second].p = i;
        node[rr.second].p = i;

        heap.push({ node[i].w, i });
    }
}

// 计算WPL
int WPL(const vector<hafu>& node, int n) {
    int wpl = 0;
    for (int i = 0; i < n; i++) {
        int d = 0;  // 深度
        int j = i;
        while (node[j].p != -1) {
            d++;
            j = node[j].p;  // 修正：应该访问当前节点j的父节点，而非i的
        }
        wpl += node[i].w * d;
    }
    return wpl;
}

```



# **搜索与图论**

## **DFS**

```c++
/*输入n，按字典序输出所有1-n的排列方式*/
#include<iostream>
using namespace std;

const int N = 10;
int n;
int path[N];
bool st[N];

void dfs(int u)
{
	if (u == n)
	{
		for (int i = 0; i < n; i++)
			printf("%d", path[i]);
		printf("\n");
		return;
	}
	for (int i = 1; i <= n; i++)
	{
		if (!st[i])
		{
			path[u] = i;
			st[i] = true;
			dfs(u + 1);
			st[i] = false;
		}
	}
}

int main()
{
	cin >> n;
	dfs(0);
	return 0;
}
```

### 1.n皇后

```c++
#include<iostream>
using namespace std;

const int N = 20;
int n;
char g[N][N];
bool col[N], dg[N], udg[N];

void dfs(int u)
{
	if (u == n)
	{
		for (int j = 0; j < n; j++) puts(g[j]);
		puts("");
		return;
	}
	for (int i = 0; i < n; i++)
	{
		if (!col[i] && !dg[u + i] && !udg[n - u + i])
		{
			g[u][i] = 'Q';
			col[i] = dg[u + i] = udg[n - u + i] = true;
			dfs(u + 1);
			col[i] = dg[u + i] = udg[n - u + i] = false;
			g[u][i] = '.';
		}
	}
}
int main()
{
	cin >> n;
	for (int j = 0; j < n; j++)
		for (int i = 0; i < n; i++)
			g[j][i] = '.';
	dfs(0);
	return 0;
}
```

### 2.树的中心

```c++
#include<iostream>
using namespace std;

const int N = 100010, M = N * 2;
int n, m;
int h[N], e[M], ne[M], idx;
bool st[N];
int ans = N;

void add(int a, int b)
{
	e[idx] = b, ne[idx] = h[a], h[a] = idx++;
}

int dfs(int u)
{
	st[u] = true;

	int sum = 1, res = 0;
	for (int i = h[u]; i != -1; i=ne[i])
	{
		int j = e[i];
		if (!st[j])
		{
			int s = dfs(j);
			res = max(res, s);
			sum += s;
		}
	}
	res = max(res, n - sum);
	ans = min(ans,res);
	return sum;
}

int main()
{
	cin >> n;
	memset(h, -1, sizeof(h));
	for (int i = 0; i < n - 1; i++)
	{
		int a, b;
		cin >> a >> b;
		add(a, b), add(b, a);
	}
	dfs(1);
	cout << ans << endl;
	return;
}
```

## **BFS**

```c++
//走迷宫
#define _CRT_SECURE_NO_WARNINGS
#include<cstring>
#include<iostream>

#include<algorithm>
using namespace std;

typedef pair<int, int> PII;
const int N = 110;
int n, m;
int g[N][N];
int d[N][N];
PII q[N * N];

int bfs()
{
	int hh = 0, tt = 0;
	q[0] = {0,0};
	memset(d, -1, sizeof(d));
	d[0][0] = 0;
	int dx[4] = { -1,0,1,0 }, dy[4] = {0,1,0,-1};
	while(hh<=tt)
	{
		auto t = q[hh++];
		for(int i=0;i<4;i++)
		{ 
			int x = t.first + dx[i], y = t.second + dy[i];
			if (x >= 0 && x < n && y >= 0 && y < m && g[x][y] == 0 && d[x][y] == -1)
			{
				d[x][y] = d[t.first][t.second] + 1;
				q[++tt] = { x,y };
			}
		}
		
	}
	return d[n - 1][m - 1];
}

int main()
{
	cin >> n >> m;
	for (int i = 0; i < n; i++)
		for (int j = 0; j < m; j++)
			scanf("%d",&g[i][j]);
	cout << bfs() << endl;
}
```

### 图重点的层次

```c++
//图，求1号点到n号点的最短距离
#include<iostream>
using namespace std;

const int N = 100010;
int n, m;
int h[N], e[N], ne[N], idx;
int d[N], q[N];

void abs(int a, int b)
{
	e[idx] = b, ne[idx] = h[a], h[a] = idx++;
}

int bfs()
{
	int hh = 0, tt = 0;
	q[0] = 1;
	memset(d, -1, sizeof(d));
	d[1] = 0;

	while (hh <= tt)
	{
		int t = q[hh++];

		for (int i = h[t]; i != -1; i = ne[i])
		{
			int j = e[i];
			if (d[j] == -1)
			{
				d[j] = d[t] + 1;
				q[++tt] = j;
			}
		}
	}
	return d[n];
}

int main()
{
	cin >> n >> m;
	memset(h, -1, sizeof h);
	for (int i = 0; i < m; i++)
	{
		int a, b;
		cin >> a >> b;
		abs(a, b);
	}
	cout << bfs() << endl;
	return 0;
}
```

## **最短路**

### **Dijkstra**

```c++
#include<iostream>
using namespace std;

const int N = 510;

int n, m;
int g[N][N];
int dist[N];
bool st[N];

int dijkstra()
{
	memset(dist, 0x3f, sizeof dist);
	dist[1] = 0;
	for (int i = 0; i < n; i++)
	{
		int t = -1;
		for (int j = 1; j <= n; j++)
			if (!st[j] && (t == -1 || dist[t] > dist[j]))
				t = j; 
		st[t] = true;
		for (int j = 1; j <= n; j++)
			dist[j] = min(dist[j], dist[t] + g[t][j]);
	}
	if (dist[n] == 0x3f3f3f3f) return -1;
	return dist[n];
}

int main()
{
	scanf("%d%d", &n, &m);
	memset(g, 0x3f, sizeof g);
	while (m--)
	{
		int a, b, c;
		scanf("%d%d%d", &a, &b, &c);
		g[a][b] = min(g[a][b], c);
	}
	int t = dijkstra();
	printf("%d\n", t);
	return 0;
}
```

### **digkstra堆优化**

```c++
#define _CRT_SECURE_NO_WARNINGS
#include<iostream>
#include<queue>
using namespace std;

typedef pair<int, int > PII;
const int N = 100010;
int n, m;
int h[N], w[N], e[N], ne[N], idx;
int dist[N];
bool st[N];

void add(int a, int b, int c)
{
	e[idx] = b, w[idx] = c, ne[idx] = h[a], h[a] = idx++;
}

int dijkstra()
{
	memset(dist, 0x3f, sizeof dist);
	dist[1] = 0;

	priority_queue<PII, vector<PII>, greater<PII>> heap;
	heap.push({ 0,1 });

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
				heap.push({dist[j],j });
			}
		}
	}
	if (dist[n] == 0x3f3f3f3f) return -1;
	return dist[n];
}

int main()
{
	scanf("%d%d", &n, &m);
	memset(h, -1, sizeof h);
	while (m--)
	{
		int a, b, c;
		scanf("%d%d%d", &a, &b, &c);
		add(a, b, c);
	}
	int t = dijkstra();
	printf("%d", t);
	return 0;
}
```

### **Bellman-Ford**

```c++
//有边数限制的最短距离k，n个点，m个边
#define _CRT_SECURE_NO_WARNINGS
#include<iostream>
#include<algorithm>
using namespace std;

const int N = 510, M = 10010;
int n, m, k;
int dist[N], backup[N];

struct Edge
{
	int a, b, w;
}edges[M];

bool bellman_ford()
{
	memset(dist, 0x3f, sizeof dist);
	dist[1] = 0;

	for (int i = 0; i < k; i++)
	{
		memcpy(backup, dist, sizeof dist);
		for (int j = 0; j < m; j++)
		{
			int a = edges[j].a, b = edges[j].b, w = edges[j].w;
			if (backup[a] != 0x3f3f3f3f)
				dist[b] = min(dist[b], backup[a] + w);
		}
	}
	return dist[n] != 0x3f3f3f3f;
}
int main()
{
	scanf("%d%d%d", &n, &m, &k);
	for (int i = 0; i < m; i++)
	{
		int a, b, w;
		scanf("%d%d%d", &a, &b, &w);
		edges[i] = { a,b,w };
	}
	if (!bellman_ford()) puts("impossible");
	else printf("%d\n", dist[n]);
	return 0;
}
```

### **SPFA**

```c++
#include<iostream>
#include<queue>
using namespace std;

const int N = 10010;
int n, m;
int h[N], e[N], w[N], ne[N], idx;
int dist[N];
bool st[N];

void add(int a, int b, int c)
{
	e[idx] = b, w[idx] = c, ne[idx] = h[a], h[a] = idx++;
}

bool spfa()
{
	memset(dist, 0x3f, sizeof dist);
	dist[1] = 0;

	queue<int> q;
	q.push(1);
	st[1] = true;
	
	while (q.size())
	{
		int t = q.front();
		q.pop();
	
		st[t] = false;
	
		for (int i = h[t]; i != -1; i = ne[i])
		{
			int j = e[i];
			if (dist[j]> dist[t] + w[i])
			{
				dist[j] = dist[t] + w[i];
				if (!st[j])
				{
					q.push(j);
					st[j] = true;
				}
			}
		}
	}
	return dist[n] != 0x3f3f3f3f;

}

int main()
{
	scanf("%d%d", &n, &m);
	memset(h, -1, sizeof h);
	while (m--)
	{
		int a, b, c;
		scanf("%d%d%d", &a, &b, &c);
		add(a, b, c);
	}
	if (!spfa()) puts("impossible");
	else printf("%d\n", dist[n]);
}
```

### **Floyd**

```c++
#include<iostream>
using namespace std;

const int N = 210, INF = 1e9;
int n, m, Q;
int d[N][N];

void Floyd()
{
	for (int k = 1; k <= n; k++)
		for (int i = 1; i <= n; i++)
			for (int j = 1; j <= n; j++)
				d[i][j] = min(d[i][j], d[i][k] + d[k][j]);
}

int main()
{
	scanf("%d%d%d", &n, &m, &Q);
	for (int i = 1; i <= n; i++)
		for (int j = 1; j <= n; j++)
			if (i == j) d[i][j] = 0;
			else d[i][j] = INF;
	while (m--)
	{
		int a, b, w;
		scanf("%d%d%d", &a, &b, &w);
		d[a][b] = w;
	}
	Floyd();
	while (Q--)
	{
		int a, b;
		scanf("%d%d", &a, &b);
		if (d[a][b] > INF / 2) printf("impossible");
		else printf("%d\n", d[a][b]);
	}
}
```

## 最小生成树

### **prim**

```c++
#define _CRT_SECURE_NO_WARNINGS
#include<iostream>
using namespace std;

const int N = 510, INF = 0x3f3f3f3f;
int n, m;
int dist[N];
int g[N][N];
bool st[N];

int prim()
{
	memset(dist, 0x3f, sizeof dist);
	int res = 0;

	for (int i = 0; i < n; i++)
	{
		int t = -1;
		for (int j = 1; j <= n; j++)
		{
			if (!st[j] && (t == -1 || dist[t] > dist[j]))
			t = j;
		}

		if (i&& dist[t] == INF) return INF;
		if (i) res += dist[t];

		for (int j = 1; j <= n; j++) 
			dist[j] = min(dist[j], g[t][j]);
		st[t] = true;
	}
	return res;
}

int main()
{
	scanf("%d%d", &n, &m);
	memset(g, 0x3f, sizeof g);
	while (m--)
	{
		int a, b, c;
		scanf("%d%d%d", &a, &b, &c);
		g[a][b] = g[b][a] = c;
	}
	int t = prim();
	if (t == INF) puts("impossible");
	else printf("%d",t);
}
```

### **kruskal**

```c++
#include<iostream>
#include<algorithm>
using namespace std;

const int N = 200010;

int n, m;
int p[N];

struct Edge
{
	int a, b, w;
	bool operator<(const Edge& W)const
	{
		return w < W.w;
	}
}edgs[N];

int find(int x)
{
	if (p[x] != x) p[x] = find(p[x]);
	return p[x];
}

int main()
{
	scanf("%d%d", &n, &m);
	for (int i = 0; i < m; i++)
	{
		int a, b, w;
		scanf("%d%d%d", &a, &b, &w);
		edgs[i] = { a,b,w };
	}
	for (int i = 1; i <= n; i++) p[i] = i;
	sort(edgs, edgs + m);

	int res=0, cnt=0;
	for (int i = 0; i < m; i++)
	{
		int a = edgs[i].a, b = edgs[i].b, w = edgs[i].w;
		a = find(a), b = find(b);
		if (a != b)
		{
			p[a] = b;
			res += w;
			cnt++;
		}
	}
	if (cnt < n - 1) puts("impossible");
	else printf("%d\n", res);
}
```

## 二分图

### **染色法**

```c++
//判断是否为二分图
#include<iostream>
using namespace std;

const int N = 100010, M = 200010;
int n, m;
int h[N], e[M], ne[M], idx;
int color[N];

void add(int a,int b)
{
	e[idx] = b, ne[idx] = h[a], h[a] = idx++;
}

bool dfs(int u, int c)
{
	color[u] = c;
	for (int i = h[u]; i != -1; i = ne[i])
	{
		int j = e[i];
		if (!color[j])
		{
			if (!dfs(j, 3 - c)) return false;
		}
		else if (color[j] == c) return false;
	}
	return true;
}

int main()
{
	scanf("%d%d", &n, &m);
	memset(h, -1, sizeof h);
	while (m--)
	{
		int a, b;
		scanf("%d%d", &a, &b);
		add(a, b), add(b, a);
	}
	bool flag = true;
	for (int i = 1; i <= n; i++)
	{
		if (!color[i])
		{
			if (!dfs(i, 1))
			{
				flag = false;
				break;
			}
		}
	}
	if (flag) puts("Yes");
	else printf("No");
	return 0;
}
```

### **匈牙利算法**

```c++
//左组n1，右组n2，m条边，找找最大匹配数量
#include<iostream>
#include<algorithm>
using namespace std;

const int N = 510, M = 100010;
int n1, n2, m;
int h[N], e[M], ne[M], idx;
int match[N];
bool st[N];

void add(int a, int b)
{
	e[idx] = b, ne[idx] = h[a], h[a] = idx++;
}

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

int main()
{
	scanf("%d%d%d", &n1, &n2, &m);
	memset(h, -1, sizeof h);

	while (m--)
	{
		int a, b;
		scanf("%d%d", &a, &b);
		add(a, b);
	}

	int res=0;
	for (int i = 1; i <= n1; i++)
	{
		memset(st, false, sizeof st);
		if (find(i)) res++;
	}
	printf("%d", res);
}
```

## **拓扑排序**

```c++
#include<iostream>
using namespace std;

const int N = 100010;
int n;
int e[N], ne[N], h[N], idx;
int rudu[N];
int q[N],hh=0,tt=-1;

void add(int a, int b)
{
	e[idx] = b, ne[idx] = h[a], h[a] = idx++;
}

int main()
{
	cin >> n;
	memset(h, -1, sizeof h);
	for (int i = 1; i <= n; i++)
	{
		while (1)
		{
			int a;
			cin >> a;
			if (a == 0)
				break;
			add(i, a);
			rudu[a]++;
		}
	}

	for (int i = 1; i <= n; i++)
	{
		if (rudu[i] == 0)
			q[++tt] = i;
	}

	while (hh <= tt)
	{
		int t = q[hh++];
		cout << t << ' ';

		for (int i = h[t]; i != -1; i = ne[i])
		{
			int j = e[i];
			rudu[j]--;
			if (rudu[j] == 0)
			{
				q[++tt] = j;
			}
		}
	}
}

```

# 数学知识

## **质数**

```c++
#define _CRT_SECURE_NO_WARNINGS
#include<iostream>
using namespace std;

const int N = 100010;

bool is_prime(int n)//判断质数
{
	if (n < 2) return false;
	for (int i = 2; i <= n / i; i++)
	{
		if (n % i == 0)
			return false;
	}
	return true;
}

void divide(int n)//分解质因数
{
	for (int i = 2; i <= n/i; i++)
	{
		if (n % i == 0)
		{
			while (n % i == 0)
			{
				n /= i;
			}
			printf("%d\n", i);
		}
	}
	if (n > 1) printf("%d\n", n);
}

int primes[N], cnt;//筛质数,cnt为数量
bool st[N];

void get_primes1(int n)//埃氏算法
{
	for (int i = 2; i < n; i++)
	{
		if (!st[i])
		{
			primes[cnt++] = i;
			for (int j = i + i; j <= n; j += i)
				st[j] = true;
		}
	}
}

void get_primes2(int n)//线性算法
{
	for (int i = 2; i <= n; i++)
	{
		if (!st[i]) primes[cnt++] = i;
		for (int j = 0; primes[j] <= n / i; j++)
		{
			st[primes[j] * i] = true;
			if (i % primes[j] == 0) break;
		}
	}
}

int main()
{
	int n;
	scanf("%d",&n);
	get_primes2(n);
	printf("%d", cnt);
}
```

## 约数

### **求约数**

```c++
#include<iostream>
#include<vector>
#include<algorithm>
using namespace std;

vector<int> get_divisors(int n)//试除法求约数
{
	vector<int> res;
	for (int i = 1; i <= n / i; i++)
	{
		if (n % i == 0)
		{
			res.push_back(i);
			if (i != n / i) res.push_back(n / i);
		}
	}
	sort(res.begin(), res.end());
	return res;
}

int main()
{
	int n;
	cin >> n;
	while (n--)
	{
		int x;
		cin >> x;
		auto res = get_divisors(x);
		for (auto t : res) cout << t << ' ';
		cout << endl;
	}
	return 0;
}
```

### **约数个数**

```c++
// n个a[i]，输出这些数乘积的约数个数，对10e9+7取模
#include<iostream>
#include<unordered_map>
using namespace std;

typedef long long LL;
const int mod = 1e9 + 7;

int main()
{
	int n;
	cin >> n;

	unordered_map<int, int> primes;
	while (n--)
	{
		int x;
		cin >> x;
		for (int i = 2; i <= x / i; i++)
			while (x % i == 0)
			{
				x /= i;
				primes[i]++;
			}
			
		if (x > 1) primes[x]++;
	}
	LL res = 1;
	for (auto prime : primes) res = res * (prime.second + 1) % mod;
	cout << res << endl;
	return 0;
}

```

### **约数之和**

```c++
#include<iostream>
#include<unordered_map>
using namespace std;

typedef long long ll;
const int mod = 10e9 + 7;

int main()
{
	int n; 
	cin >> n;
	unordered_map<int, int> primes;
	while (n--)
	{
		int x; 
		cin >> x;
		for (int i = 2; i <= x / i; i++)
			while (x % i == 0)
			{
				x /= i;
				primes[i]++;
			}
		if (x > 1) primes[x]++;
	}

	ll res = 1;
	for (auto prime : primes)
	{
		int p = prime.first, a = prime.second;
		ll t = 1;
		while (a--)
			t = (t * p + 1) % mod;
		res = res * t % mod;
	}
	cout << res << endl;
}

```

## **欧几里得算法（求最大公约数）**

```c++
#include<iostream>
using namespace std;

int gcd(int a, int b)
{
	return b ? gcd(b, a % b) : a;
}
int main()
{
	int n; cin >> n;
	while (n--)
	{
		int a, b;
		cin >> a >> b;
		printf("%d", gcd(a, b));
	}
	return 0;
}
```

## **快速幂**

```c++
#include<iostream>
using namespace std;

typedef long long ll;

int qmi(int a,int b,int p)
{
	int res = 1;
	while (b)
	{
		if (b & 1) res = (ll)res * a % p;
		b >>= 1;
		a = (ll)a * a % p;
	}
	return res;
}

int main()
{
	int a, b, p;
	cin >> a >> b >> p;
	cout << qmi(a, b, p);
}
```

## **扩展欧几里得算法**

```c++
#include<iostream>
using namespace std;

int exgcd(int a, int b, int& x, int& y)
{
	if (!b)
	{
		x = 1, y = 0;
		return a;
	}
	int d = exgcd(b, a % b, y, x);
	y -= a / b * x;
	return d;
}

int main()
{
	int n;
	cin >> n;
	while (n--)
	{
		int a, b, x, y;
		cin >> a >> b;
		exgcd(a, b, x, y);
		cout << x << ' ' << y;
	}
}
```

## 线性同余方程

```c++
#include<iostream>
using namespace std;

int exgcd(int a, int b, int& x, int& y)
{
	if (!b)
	{
		x = 1, y = 0;
		return a;
	}
	else
	{
		int d = exgcd(b, a % b, y, x);
		y -= a / b * x;
		return d;
	}
}

int main()
{
	int n;
	cin >> n;
	while (n--)
	{
		int a, b, m;
		cin >> a >> b >> m;
		int x, y;
		int d = exgcd(a, m, x, y);
		if (b % d) puts("impossible");
		else cout << x * (b / d) % m;
	}
}
```

## 高斯消元求线性方程组

```c++
#include<iostream>
#include<cmath>
using namespace std;

const int N = 110;
const double eps = 1e-6;

int n;
double a[N][N];

int gauss()
{
	int c, r;
	for (c = 0, r = 0; c < n; c++)
	{
		int t = r;
		for (int i = r; i < n; i++)
			if (fabs(a[i][c]) > fabs(a[t][c]))//找每列最大值
				t = i;
		if (fabs(a[t][c]) < eps) continue;

		for (int i = c; i <= n; i++) swap(a[t][i], a[r][i]);//与首行交换
		for (int i = n; i >= c; i--) a[r][i] /= a[r][c];//首列第一个元素改为1
		for (int i = r + 1; i < n; i++)//除了首元变为0
			if (fabs(a[i][c]) > eps)
				for (int j = n; j >= c; j--)
					a[i][j] -= a[r][j] * a[i][c];

		r++;
	}
	if (r < n)
	{
		for (int i = r; i < n; i++)
			if (fabs(a[i][n]) > eps)
				return 2;
		return 1;
	}
	for (int i = n - 1; i >= 0; i--)
		for (int j = i + 1; j < n; j++)
			a[i][n] -= a[i][j] * a[j][n];
	return 0;
}

int main()
{
	cin >> n;
	for (int i = 0; i < n; i++)
		for (int j = 0; j < n+1; j++)
			cin >> a[i][j];

	int t = gauss();
	if (t == 0)
		for (int i = 0; i < n; i++)
			cout << a[i][n] << endl;
	else if (t == 1)
		cout << "Infinite group solutions" << endl;
	else
		cout << "No solution" << endl;
}
```

## 组合数

### 求组合数1

（n<=100000,1<=b<=a<=2000）递推

```c++
#include<iostream>
using namespace std;

const int N = 2010, mod = 1e9 + 7;
int c[N][N];

void init()
{
	for (int i = 0; i < N; i++)
		for (int j = 0; j <= i; j++)
			if (!j) c[i][j] = 1;
			else c[i][j] = (c[i - 1][j] + c[i - 1][j - 1]) % mod;
}
int main()
{
	init();
	int n;
	cin >> n;
	while (n--)
	{
		int a, b;
		cin >> a >> b;
		cout << c[a][b] << endl;
	}
}
```

### 求组合数2

（n<=10000,1<=b<=a<=1e5）预处理

```c++
#include<iostream>
using namespace std;

typedef long long LL;
const int N = 100010, mod = 1e9 + 7;
int fact[N], infact[N];

int qmi(int a, int k, int p)
{
	int res = 1;
	while (k)
	{
		if (k & 1) res = (LL)res * a % p;
		a = (LL)a * a % p;
		k >>=1;
	}
	return res;
}

int main()
{
	fact[0] = infact[0] = 1;
	for (int i = 1; i < N; i++)
	{
		fact[i] = (LL)fact[i - 1] * i % mod;
		infact[i] = (LL)infact[i - 1] * qmi(i, mod - 2, mod) % mod;
	}
	int n;
	cin >> n;
	while (n--)
	{
		int a, b;
		cin >> a >> b;
		cout << (LL)fact[a] * infact[b] % mod * infact[a - b] % mod<<endl;
	}
}
```

### 求组合数3

（n<=20,1<=b<=a<=1e18，1<=p<=1e5(模)）卢卡斯定理

```c++
#include<iostream>
using namespace std;

typedef long long LL;
int p;

int qmi(int a, int k)
{
	int res = 1;
	while (k)
	{
		if(k&1)res = (LL)res * a % p;
		a = (LL)a * a % p;
		k >>= 1;
	}
	return res;
}

int C(int a, int b)
{
	int res = 1;
	for (int i = 1, j = a; i <= b; i++, j--)
	{
		res = (LL)res * j % p;
		res = (LL)res * qmi(i, p - 2) % p;
	}
	return res;
}

int lucas(LL a, LL b)
{
	if (a < p && b < p) return C(a, b);
	return (LL)C(a % p, b % p) * lucas(a / p, b / p) % p;
}

int main()
{
	int n;
	cin >> n;
	while (n--)
	{
		LL a, b;
		cin >> a >> b >> p;
		cout << lucas(a, b) << endl;
	}
	return 0;
}
```

### 求组合数4

（1<=b<=a<=5000 不取模）筛质数，高精度乘法 

```c++
#include<iostream>
#include<algorithm>
#include<vector>
using namespace std;

const int N = 5010;
int primes[N], cnt;
int sum[N];
bool st[N];

void get_primes(int n)
{
	for (int i = 2; i <= n; i++)
	{
		if (!st[i]) primes[cnt++] = i;
		for (int j = 0; primes[j] <= n / i; j++)
		{
			st[primes[j] * i] = true;
			if (i % primes[j] == 0) break;
		}
	}
}

int get(int n, int p)
{
	int res = 0;
	while (n)
	{
		res += n / p;
		n /= p;
	}
	return res;
}

vector<int> mul(vector<int> a, int b)
{
	vector<int> c;
	int t = 0;
	for (int i = 0; i < a.size(); i++)
	{
		t += a[i] * b;
		c.push_back(t % 10);
		t /= 10;
	}

	while (t)
	{
		c.push_back(t % 10);
		t /= 10;
	}

	return c;
}

int main()
{
	int a, b;
	cin >> a >> b;
	get_primes(a);

	for (int i = 0; i < cnt; i++)
	{
		int p = primes[i];
		sum[i] = get(a, p) - get(b, p) - get(a - b, p);
	}

	vector<int> res;
	res.push_back(1);
	for (int i = 0; i < cnt; i++)
		for (int j = 0; j < sum[i]; j++)
			res = mul(res, primes[i]);

	for (int i = res.size() - 1; i >= 0; i--)
		cout << res[i];
}
```

## 卡特兰数

```c++
#include<iostream>
using namespace std;

typedef long long LL;
const int mod = 1e9 + 7;

int qmi(int a,int k,int p)
{
	int res = 1;
	while (k)
	{
		if (k & 1) res = (LL)res * a % p;
		a = (LL)a * a % p;
		k >>= 1;
	}
	return res;
}

int main()
{
	int n;
	cin >> n;
	int a = 2 * n, b = n;
	int res = 1;
	for (int i = a; i > a - b; i--) res = (LL)res * i % mod;
	for (int i = b; i > 0; i--) res = (LL)res * qmi(i, mod - 2, mod)%mod;
	res = (LL)res * qmi(n + 1, mod - 2, mod) % mod;
	cout << res << endl;
}
```



# **贪心**

## **合并果子**

```c++
#include<iostream>
#include<algorithm>
#include<queue>
using namespace std;

int main()
{
	int n;
	scanf("%d", &n);

	priority_queue<int, vector<int>, greater<int>> heap;
	while (n--)
	{
		int x;
		scanf("%d", &x);
		heap.push(x);
	}
	int res = 0;
	while (heap.size() > 1)
	{
		int a = heap.top(); heap.pop();
		int b = heap.top(); heap.pop();
		res += a + b;
		heap.push(a + b);
	}
	printf("%d\n", res);
	return 0;
}
```

## **区间覆盖**

```c++
//给出一个区间，求覆盖这个区间所用区间最小值
#include<iostream>
#include<algorithm>
using namespace std;

const int N = 100010;
int n;
struct Range
{
	int l, r;
	bool operator < (const Range& W)const
	{
		return l < W.l;
	}
}range[N];

int main()
{
	int st, ed;
	scanf("%d%d", &st, &ed);
	scanf("%d", &n);
	for (int i = 0; i < n; i++)
	{
		int l, r;
		scanf("%d%d", &l, &r);
		range[i] = { l,r };
	}

	sort(range, range + n);

	int res = 0;
	bool success = false;
	for (int i = 0; i < n; i++)
	{
		int j = i, r = -2e9;
		while (j < n && range[j].l <= st)
		{
			r = max(r, range[j].r);
			j++;
		}
		if (r < st)
		{
			res = -1;
			break;
		}
		res++;
		if (r >= ed)
		{
			success = true;
			break;
		}

		st = r;
		i = j - 1;
	}
	if (!success) res = -1;
	printf("%d\n", res);
}
```

## **区间分组**

```c++
//将区间分组，每组其中不含重叠区间，最少多少组
#include<iostream>
#include<algorithm>
#include<queue>
using namespace std;

const int N = 100010;
int n;
struct Range
{
	int l, r;
	bool operator<(const Range& W) const
	{
		return l < W.l;
	}
}range[N];

int main()
{
	scanf("%d", &n);
	for (int i = 0; i < n; i++)
	{
		int l, r;
		scanf("%d%d", &l, &r);
		range[i] = { l,r };
	}

	sort(range, range + n);

	priority_queue<int, vector<int>, greater<int>>heap;
	for (int i = 0; i < n; i++)
	{
		auto r = range[i];
		if (heap.empty() || heap.top() >= r.l) heap.push(r.r);
		else
		{
			int t = heap.top();
			heap. pop();
			heap.push(r.r);
		}
	}
	printf("%d\n", heap.size());
	return 0;
}
```

## **区间选点**

```c++
//输入若干个区间，取点使所有区间内都有点，求点的最少数量
#include<iostream>
#include<algorithm>
using namespace std;

const int N = 100010;
int n;
struct Range
{
	int l, r;
	bool operator<(const Range& W)const
	{
		return r < W.r;
	}
}range[N];

int main()
{
	scanf("%d", &n);
	for(int i=0;i<n;i++)
	{
		int l, r;
		scanf("%d%d", &l, &r);
		range[i] = { l,r };
	}
	sort(range, range + n);

	int res = 0, ed = -2e9;
	for (int i = 0; i < n; i++)
	{
		if (range[i].l > ed)
		{
			res++;
			ed = range[i].r;
		}
	}

	printf("%d", res);
}

```

# **动态规划**

## 背包

### **01背包**

```c++
#include<iostream>
using namespace std;

const int N = 1010;
int n, m;
int v[N], w[N];
int f[N][N];

int main()
{
	cin >> n >> m;
	for (int i = 1; i <= n; i++) cin >> v[i] >> w[i];

	for(int i=1;i<=n;i++)
		for (int j = 0; j <= m; j++)
		{
			f[i][j] = f[i - 1][j];
			if (j >= v[i]) f[i][j] = max(f[i][j], f[i - 1][j - v[i]] + w[i]);
		}
	cout << f[n][m];
}
```

#### **一维**

```c++
#include<iostream>
using namespace std;

const int N = 1010;
int n, m;
int v[N], w[N];
int f[N];

int main()
{
	cin >> n >> m;
	for (int i = 1; i <= n; i++) cin >> v[i] >> w[i];

	for (int i = 1; i <= n; i++)
		for (int j = m; j >= v[i]; j--)
			f[j] = max(f[j], f[j - v[i]] + w[i]);
	cout <<f[m];
}
```

### **完全背包**

```c++
#include<iostream>
using namespace std;

const int N = 1010;
int n, m;
int v[N], w[N];
int f[N][N];

int main()
{
	cin >> n >> m;
	for (int i = 1; i <= n; i++) cin >> v[i] >> w[i];
	for (int i = 1; i <= n; i++)
		for (int j = 0; j <= m; j++)
			for (int k = 0; k * v[i] <= j; k++)
				f[i][j] = max(f[i][j], f[i - 1][j - v[i] * k] + w[i] * k);
	cout << f[n][m];
}
```

#### **(优化)**

```c++
#include<iostream>
using namespace std;


const int N = 1010;
int n, m;
int v[N], w[N];
int f[N][N];

int main()			优化
{
	cin >> n >> m;
	for (int i = 1; i <= n; i++) 
		cin >> v[i] >> w[i];

	for (int i = 1; i <= n; i++)
		for (int j = 0; j <= m; j++)
		{
			f[i][j] = f[i - 1][j];
			if (j >= v[i]) f[i][j] = max(f[i][j], f[i][j - v[i]] + w[i]);

		}
	cout << f[n][m];
}


*****************************************************************************
const int N = 1010;
int n, m;
int v[N], w[N];
int f[N];

int main()	//再优化为一维
{
	cin >> n >> m;
	for (int i = 1; i <= n; i++)
		cin >> v[i] >> w[i];

	for (int i = 1; i <= n; i++)
		for (int j = v[i]; j <= m; j++)
			f[j] = max(f[j], f[j - v[i]] + w[i]);
	cout << f[m];
}
```

### **多重背包**

```c++
#include<iostream>
using namespace std;

const int N = 110;
int n, m;
int v[N], w[N],s[N];
int f[N][N];

int main()
{
	cin >> n >> m;
	for (int i = 1; i <= n; i++) cin >> v[i] >> w[i]>>s[i];

	for (int i = 1; i <= n; i++)
		for (int j = 0; j <= m; j++)
			for (int k = 0;k<=s[i]&& k * v[i] <= j; k++)
				f[i][j] = max(f[i][j], f[i - 1][j - v[i] * k] + w[i] * k);

	cout << f[n][m] << endl;
	return 0;
}
```

#### **优化**

```c++
#include<iostream>
using namespace std;

const int N = 250000;
int n, m;
int v[N], w[N], s[N];
int f[N];

int main()
{
	cin >> n >> m;

	int cnt = 0;
	for (int i = 1; i <= n; i++)
	{
		int a, b, s;
		cin >> a >> b >> s;
		int k = 1;
		while (k <= s)
		{
			cnt++;
			v[cnt] = a * k;
			w[cnt] = b * k;
			s -= k;
			k *= 2;
		}
		if (s > 0)
		{
			cnt++;
			v[cnt] = a * s;
			w[cnt] = b * s;
		}
	}

	n = cnt;
	for (int i = 1; i <= n; i++)
		for (int j = m; j >= v[i]; j--)
			f[j] = max(f[j], f[j - v[i]] + w[i]);
	cout << f[m];
	return 0;
}
```

### **分组背包**

```c++
#include<iostream>
using namespace std;

const int N = 110;
int n, m;
int v[N][N], w[N][N],s[N];
int f[N];

int main()
{
	cin >> n >> m;

	for (int i = 1; i <= n; i++)
	{
		cin >> s[i];
		for (int j = 0; j < s[i]; j++)
			cin >> v[i][j] >> w[i][j];
	}

	for (int i = 1; i <= n; i++)
		for (int j = m; j >= 0; j--)
			for (int k = 0; k < s[i]; k++)
				if (v[i][k] <= j)
					f[j] = max(f[j], f[j - v[i][k]] + w[i][k]);

	cout << f[m] << endl;
	return 0;
}
```

## **线性DP**

```c++
#include<iostream>
using namespace std;

const int N = 510, INF = 1e9;
int n;
int a[N][N];
int f[N][N];

int main()
{
	cin >> n;
	for (int i = 1; i <= n; i++)
		for (int j = 1; j <= i; j++)
			cin >> a[i][j];
	for (int i = 0; i <= n; i++)
		for (int j = 0; j <= i + 1; j++)
			f[i][j] = -INF;
	f[1][1] = a[1][1];

	for (int i = 2; i <= n; i++)
		for (int j = 1; j <= i; j++)
			f[i][j] = max(f[i - 1][j - 1] + a[i][j], f[i - 1][j] + a[i][j]);
	int res = -INF;
	for (int i = 1; i <= n; i++)
		res = max(res, f[n][i]);
	cout << res;
}
```

## **最长上升子序列**

```c++
#include<iostream>
using namespace std;

const int N = 1010;
int n;
int a[N], f[N],g[N];

int main()
{
	cin >> n;
	for (int i = 1; i <= n; i++) cin >> a[i];
	for (int i = 1; i <= n; i++)
	{
		f[i] = 1;
		g[i] = 0;
		for (int j = 1; j < i; j++)       //记录最长子串
			if(a[j]<a[i])
				if (f[i] < f[j] + 1)
				{
					f[i] = f[j] + 1;
					g[i] = j;
				}
	}
	int k = 1;
	for (int i = 1; i <= n; i++)
		if (f[k] < f[i])
			k = i;
	cout << f[k];
	for (int i = 0, len = f[k]; i < len; i++)
	{
		cout << a[k];
		k = g[k];
	}
	return 0;
}

/*
for (int i = 1; i <= n; i++)
	{
		f[i] = 1;
		for (int j = 1; j < i; j++)
			if (a[i] > a[j])
				f[i] = max(f[j] + 1, f[i]);
	}
	int res = 0;
	for (int i = 1; i <= n; i++)
		res = max(res, f[i]);
	cout << res;
*/
```

## **最长公共子序列**

```c++
#include<iostream>
using namespace std;

const int N = 1010;

int n, m;
char a[N], b[N];
int f[N][N];

int main()
{
	cin >> n >> m;
	scanf("%s%s", a + 1, b + 1);

	for (int i = 1; i <= n; i++)
		for (int j = 1; j <= m; j++)
		{
			f[i][j] = max(f[i - 1][j], f[i][j - 1]);
			if (a[i] == b[j]) f[i][j] = max(f[i][j], f[i - 1][j - 1] + 1);
		}
	cout << f[n][m];
}
```

## **区间DP(石子合并)**

```c++
#include<iostream>
using namespace std;

const int N = 310;
int n;
int s[N];
int f[N][N];

int main()
{
	cin >> n;
	for (int i = 1; i <= n; i++)
		cin >> s[i];

	for (int i = 1; i <= n; i++) s[i] += s[i - 1];

	for (int len = 2; len <= n; len++)
	{
		for (int i = 1; i + len - 1 <= n; i++)
		{
			int l = i, r = i + len - 1;
			f[l][r] = 1e9;
			for (int k = l; k < r; k++)
			{
				f[l][r] = min(f[l][r], f[l][k] + f[k + 1][r] + s[r] - s[l - 1]);
			}
		}
	}
	cout << f[1][n];
}
```

# 高精度

## 高精度乘法

```c++
#include<iostream>
#include<vector>
using namespace std;
//C=A*b
vector<int> mul(vector<int>& A,int b)
{
    vector<int>C;
    int t = 0;//进位
    for (int i = 0; i < A.size()||t; i++)
    {
        if (i < A.size())t += A[i] * b;
        C.push_back(t % 10);
        t /= 10;
    }
    return C;
}
int main()
{
    string a;
    vector<int>A;
    int b;
    cin >> a >> b;

    for (int i = a.size() - 1; i >= 0; i--)
        A.push_back(a[i] - '0');
    
    auto C = mul (A, b);
    
    for (int i = C.size() - 1; i >= 0; i--)
        printf("%d", C[i]);
    return 0;

}
```

------

## 加法

```c++
#include<iostream>
#include<vector>
using namespace std;
C=A+B；
vector<int> add(vector<int>& A, vector<int>& B)
{
    vector<int> C;
    if (A.size() < B.size()) return add(B, A);
    int t = 0;
    for (int i = 0; i < A.size(); i++)
    {
        t += A[i];
        if (i < B.size()) t += B[i];
        C.push_back(t % 10);
        t /= 10;
    }

    if (t) C.push_back(1);
    return C;

}
int main()
{
    string a, b;
    vector<int>A, B;

    cin >> a >> b;
    for (int i = a.size()-1; i >= 0; i--)
        A.push_back(a[i] - '0');
    for (int i = b.size()-1; i >= 0; i--)
        B.push_back(b[i] - '0');
    
    auto C = add(A, B);
    
    for (int i = C.size() - 1; i >= 0; i--)
        printf("%d", C[i]);
    return 0;

}
```

------

## 减法

```c++
#include<iostream>
#include<vector>
using namespace std;
//判断是否有A>B
bool cmp(vector<int>A, vector<int>B)
{
    if (A.size() != B.size())
        return A.size() > B.size();
    for (int i = A.size() - 1; i >= 0; i--)
        if (A[i] != B[i])
            return A[i] > B[i];
    return true;
}
//C=A-B
vector<int> sub(vector<int>& A, vector<int>& B)
{
    vector<int>C;
    for (int i = 0, t = 0; i < A.size(); i++)
    {
        t = A[i] - t;
        if (i < B.size()) t -= B[i];
        C.push_back((t + 10) % 10);
        if (t < 0) t = 1;
        else t = 0;
    }
    while (C.size() > 1 && C.back() == 0)//去掉前导0
        C.pop_back();
    return C;
}
int main()
{
    string a, b;
    vector<int>A, B;

    cin >> a >> b;
    for (int i = a.size() - 1; i >= 0; i--)
        A.push_back(a[i] - '0');
    for (int i = b.size() - 1; i >= 0; i--)
        B.push_back(b[i] - '0');
    
    if (cmp(A, B))
    {
        auto C = sub(A, B);
        for (int i = C.size() - 1; i >= 0; i--)
            printf("%d", C[i]);
    }
    else
    {
        auto C = sub(B, A);
        printf("-");
        for (int i = C.size() - 1; i >= 0; i--)
            printf("%d", C[i]);
    }
    return 0;

}
```

## **除法**

```c++
#include<iostream>
#include<vector>
#include<algorithm>
using namespace std;
//A/b,商是C,余数是r
vector<int> div(vector<int>& A, int b,int &r)
{
    vector<int>C;
    r = 0;
    for (int i =A.size()-1; i>=0; i--)
    {
        r = r*10 + A[i];
        C.push_back(r / b);
        r %= b;
    }
    reverse(C.begin(), C.end());
    while (C.size() > 1 && C.back() == 0)
        C.pop_back();
    return C;
}
int main()
{
    string a;
    vector<int>A;
    int b;
    cin >> a >> b;

    for (int i = a.size() - 1; i >= 0; i--)
        A.push_back(a[i] - '0');
    int r;
    auto C = div(A, b,r);

    for (int i = C.size() - 1; i >= 0; i--)
        printf("%d", C[i]);
    cout << endl << r << endl;
    return 0;
}
```

# 杂项

# 莫队

## **普通莫队**

```c++
#include<iostream>
#include<algorithm>
using namespace std;

// 普通莫队入门题
// 给定一个长度为n的数组arr，一共有q条查询，格式如下
// 查询 l r : 打印arr[l..r]范围上有几种不同的数字
// 1 <= n <= 3 * 10^4
// 1 <= arr[i] <= 10^6
// 1 <= q <= 2 * 10^5

struct Query {
    int l, r, id;
};

const int MAXN = 30001;
const int MAXV = 1000001;
const int MAXQ = 200001;
int n, q;
int arr[MAXN];
Query query[MAXQ];

int bi[MAXN];//每个数所在的块
int cnt[MAXV];
int kind = 0;

int ans[MAXQ];

//经典排序策略
bool QueryCmp1(Query &a, Query &b) {
    if (bi[a.l] != bi[b.l]) {
        return bi[a.l] < bi[b.l]; //块从小到大
    }
    return a.r < b.r;//块一样，jobr从小到大
}
//奇偶排序策略
bool QueryCmp2(Query &a, Query &b) {
    if (bi[a.l] != bi[b.l]) {
        return bi[a.l] < bi[b.l];
    }
    if ((bi[a.l] & 1) == 1) {
        return a.r < b.r;
    } else {
        return a.r > b.r;
    }
}

void del(int num) {
    if (--cnt[num] == 0) {
        kind--;
    }
}

void add(int num) {
    if (++cnt[num] == 1) {
        kind++;
    }
}

void prepare() {
    int blen = (int)sqrt(n);
    for (int i = 1; i <= n; i++) {
        bi[i] = (i - 1) / blen + 1;
    }
    sort(query + 1, query + q + 1, QueryCmp2);
}

//窗口滑动
void compute() {
    int winl = 1, winr = 0;
    for (int i = 1; i <= q; i++) {
        int jobl = query[i].l;
        int jobr = query[i].r;
        while (winl > jobl) {
            add(arr[--winl]);
        }
        while (winr < jobr) {
            add(arr[++winr]);
        }
        while (winl < jobl) {
            del(arr[winl++]);
        }
        while (winr > jobr) {
            del(arr[winr--]);
        }
        ans[query[i].id] = kind;
    }
}

int main() {
    cin >> n;
    for (int i = 1; i <= n; i++) {
        cin >> arr[i];
    }
    cin >> q;
    for (int i = 1; i <= q; i++) {
        cin >> query[i].l;
        cin >> query[i].r;
        query[i].id = i;
    }
    prepare();
    compute();
    for (int i = 1; i <= q; i++) {
        cout << ans[i] << '\n';
    }
    return 0;
}
```

## 带修莫队

```c++
#include<iostream>
#include<algorithm>
#include<cmath>
using namespace std;

// 带修莫队入门题，C++版
// 给定一个长度为n的数组arr，一共有m条操作，操作格式如下
// 操作 Q l r     : 打印arr[l..r]范围上有几种不同的数
// 操作 R pos val : 把arr[pos]的值设置成val
// 1 <= n、m <= 2 * 10^5
// 1 <= arr[i]、val <= 10^6

struct Query {
    int l, r, t, id;
};

struct Update {
    int pos, val;
};

const int MAXN = 200001;
const int MAXV = 1000001;
int n, m;
int arr[MAXN];//原数组
int bi[MAXN];//分块

//可离线数据
Query query[MAXN];//查询
Update update[MAXN];//修改
int cntq, cntu;

int cnt[MAXV];//窗口内前缀异或和为x的数量
int kind;

int ans[MAXN];

//排序
bool QueryCmp(Query &a, Query &b) {
    if (bi[a.l] != bi[b.l]) {
        return bi[a.l] < bi[b.l];
    }
    if (bi[a.r] != bi[b.r]) {
        return bi[a.r] < bi[b.r];
    }
    return a.t < b.t;
}

void del(int num) {
    if (--cnt[num] == 0) {
        kind--;
    }
}

void add(int num) {
    if (++cnt[num] == 1) {
        kind++;
    }
}
//进行第tim的修改，并作用到待求区间
void moveTime(int jobl, int jobr, int tim) {
    int pos = update[tim].pos;
    int val = update[tim].val;
    if (jobl <= pos && pos <= jobr) {
        del(arr[pos]);
        add(val);
    }
    int tmp = arr[pos];
    arr[pos] = val;
    update[tim].val = tmp;
}

void compute() {
    int winl = 1, winr = 0, wint = 0;
    for (int i = 1; i <= cntq; i++) {
        int jobl = query[i].l;
        int jobr = query[i].r;
        int jobt = query[i].t;
        int id = query[i].id;
        //先进行区间移动
        while (winl > jobl) {
            add(arr[--winl]);
        }
        while (winr < jobr) {
            add(arr[++winr]);
        }
        while (winl < jobl) {
            del(arr[winl++]);
        }
        while (winr > jobr) {
            del(arr[winr--]);
        }
        //查看是否要修改
        while (wint < jobt) {
            moveTime(jobl, jobr, ++wint);
        }
        while (wint > jobt) {
            moveTime(jobl, jobr, wint--);
        }
        ans[id] = kind;
     }
}
//分块
void prepare() {
    int blen = max(1, (int)pow(n, 2.0 / 3));
    for (int i = 1; i <= n; i++) {
        bi[i] = (i - 1) / blen + 1;
    }
    sort(query + 1, query + cntq + 1, QueryCmp);
}

int main() {
    ios::sync_with_stdio(false);
    cin.tie(nullptr);
    cin >> n >> m;
    for (int i = 1; i <= n; i++) {
        cin >> arr[i];
    }
    char op;
    int l, r, pos, val;
    for (int i = 1; i <= m; i++) {
        cin >> op;
        if (op == 'Q') {
            cin >> l >> r;
            cntq++;
            query[cntq].l = l;
            query[cntq].r = r;
            query[cntq].t = cntu;
            query[cntq].id = cntq;
        } else {
            cin >> pos >> val;
            cntu++;
            update[cntu].pos = pos;
            update[cntu].val = val;
        }
    }
    prepare();
    compute();
    for (int i = 1; i <= cntq; i++) {
        cout << ans[i] << '\n';
    }
    return 0;
}
```

