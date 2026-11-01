# Python 算法模板

与 [C++ 模板](./CppTem.md) 按知识点对应的 Python 实现。导航：[README](./README.md) · [C++](./CppTem.md) · [Java](./JavaTem.md) · [Python](./PythonTem.md) · [Go](./GoTem.md) · [TypeScript](./TypeScriptTem.md) · [Rust](./RustTem.md)。

## 使用约定

- 本文按知识点提供函数与数据结构；根据题目补输入输出，不把所有示例当作一个题目的解答。
- 未特别说明时，数组和图节点从 **0** 开始；每节单独说明闭区间或左闭右开区间。树状数组使用 **1** 起始下标。
- 输入须满足各节列出的前提与有效下标。Java 的和、距离及价值使用 `long`，仍需按题目核对范围；Python 整数可扩展，但算法复杂度和内存开销依然存在。
- Python 的深递归会受递归深度限制；图遍历、树形 DP 与增广路采用迭代写法。递归搜索适用于深度较小的题。

## 内容导航

[基础算法](#基础算法) · [数据结构](#数据结构) · [搜索与图论](#搜索与图论) · [数学知识](#数学知识) · [贪心](#贪心) · [动态规划](#动态规划) · [高精度](#高精度) · [莫队](#莫队)

# 基础算法

## 快速输入

Java 的 FastScanner 放在 Main 类内；到达输入末尾时 next() 返回 null。Python 按需读取，下面演示整数输入。

```python
import sys

def read_ints():
    return list(map(int, sys.stdin.buffer.read().split()))
```

## 排序：快速排序与归并排序

原地修改数组。快速排序使用中点枢轴，最坏 O(n²)；归并排序 O(n log n)，需要 O(n) 辅助空间。

```python
def quick_sort(a, l=0, r=None):
    if r is None:
        r = len(a) - 1
    # 显式栈避免 Python 的递归深度限制。
    stack = [(l, r)]
    while stack:
        l, r = stack.pop()
        if l >= r:
            continue
        i, j, x = l - 1, r + 1, a[(l + r) // 2]
        while i < j:
            i += 1
            while a[i] < x:
                i += 1
            j -= 1
            while a[j] > x:
                j -= 1
            if i < j:
                a[i], a[j] = a[j], a[i]
        stack.extend(((l, j), (j + 1, r)))

def merge_sort(a):
    tmp = [0] * len(a)
    def sort(l, r):
        if r - l <= 1:
            return
        m = (l + r) // 2
        sort(l, m)
        sort(m, r)
        i, j, k = l, m, l
        while i < m and j < r:
            if a[i] <= a[j]:
                tmp[k] = a[i]
                i += 1
            else:
                tmp[k] = a[j]
                j += 1
            k += 1
        tmp[k:r] = a[i:m] + a[j:r]
        a[l:r] = tmp[l:r]
    sort(0, len(a))
```

## 前缀和：一维与二维

查询采用左闭右开区间 [l,r) 和矩形 [x1,x2) × [y1,y2)。构建 O(n) / O(nm)，查询 O(1)。

```python
def prefix(a):
    s = [0]
    for x in a:
        s.append(s[-1] + x)
    return s  # [l,r) 的和：s[r] - s[l]

def prefix_2d(a):
    n, m = len(a), len(a[0]) if a else 0
    s = [[0] * (m + 1) for _ in range(n + 1)]
    for i in range(n):
        for j in range(m):
            s[i + 1][j + 1] = a[i][j] + s[i][j + 1] + s[i + 1][j] - s[i][j]
    return s

def rect_sum(s, x1, y1, x2, y2):
    return s[x2][y2] - s[x1][y2] - s[x2][y1] + s[x1][y1]
```

## 差分：一维与二维

对零数组批量区间加，最后统一还原。更新采用左闭右开区间，二维差分预留一行一列边界。

```python
def range_add(d, l, r, v):
    d[l] += v
    d[r] -= v

def restore(d):
    a, total = [], 0
    for x in d[:-1]:
        total += x
        a.append(total)
    return a

def rect_add(d, x1, y1, x2, y2, v):
    d[x1][y1] += v
    d[x2][y1] -= v
    d[x1][y2] -= v
    d[x2][y2] += v

def restore_2d(d):
    n, m = len(d) - 1, len(d[0]) - 1
    a = [[0] * m for _ in range(n)]
    for i in range(n):
        for j in range(m):
            a[i][j] = (d[i][j] + (a[i - 1][j] if i else 0)
                       + (a[i][j - 1] if j else 0)
                       - (a[i - 1][j - 1] if i and j else 0))
    return a
```

## 二分：整数边界与浮点数

有序数组返回第一个 ≥ x / > x 的下标，不存在时为 n。浮点二分示例求非负数平方根，固定迭代次数。

```python
from bisect import bisect_left, bisect_right
import math

def lower_bound(a, x):
    return bisect_left(a, x)

def upper_bound(a, x):
    return bisect_right(a, x)

def sqrt_binary(x):
    if not math.isfinite(x) or x < 0:
        raise ValueError('x must be finite and nonnegative')
    l, r = 0.0, max(1.0, x)
    for _ in range(1100):
        m = l + (r - l) / 2
        if m == 0 or m <= x / m:
            l = m
        else:
            r = m
    return l
```

## 位运算

lowbit 用于正整数；统计 1 的个数时 Java 按 32 位补码，Python 版本限定非负整数。

```python
def lowbit(x):
    return x & -x

def popcount(x):
    if x < 0:
        raise ValueError('x must be nonnegative')
    count = 0
    while x:
        x &= x - 1
        count += 1
    return count
```

## 区间合并

闭区间，端点相接视为重叠。返回新数组，排序 O(n log n)。

```python
def merge_intervals(ranges):
    res = []
    for l, r in sorted(ranges):
        if not res or res[-1][1] < l:
            res.append([l, r])
        else:
            res[-1][1] = max(res[-1][1], r)
    return res
```

## 离散化

返回排序去重值及原数组对应的零基排名；排序 O(n log n)。

```python
def compress(a):
    values = sorted(set(a))
    rank = {x: i for i, x in enumerate(values)}
    return values, [rank[x] for x in a]
```

# 数据结构

## 数组模拟单链表与双链表

插入返回稳定的节点编号；删除只摘链，不复用编号。单链表用 0 作头哨兵；双链表用 0、1 作左右哨兵。容量参数是最多插入次数，O(1) 更新。

```python
class SingleList:
    def __init__(self):
        self.value, self.next = [None], [-1]

    def insert_after(self, k, x):
        i = len(self.value)
        self.value.append(x)
        self.next.append(self.next[k])
        self.next[k] = i
        return i

    def remove_after(self, k):
        if self.next[k] != -1:
            self.next[k] = self.next[self.next[k]]

class DoubleList:
    def __init__(self):
        self.value, self.left, self.right = [None, None], [-1, 0], [1, -1]

    def insert_after(self, k, x):
        i = len(self.value)
        self.value.append(x)
        self.left.append(k)
        self.right.append(self.right[k])
        self.left[self.right[k]] = i
        self.right[k] = i
        return i

    def remove(self, k):
        self.right[self.left[k]] = self.right[k]
        self.left[self.right[k]] = self.left[k]
```

## 栈、队列、堆与标准集合

对应 C++ STL。Java 优先用 ArrayDeque；堆操作 O(log n)，哈希查询期望 O(1)，有序集合 O(log n)。比较器使用 compare，避免相减溢出。

```python
from collections import deque, Counter, defaultdict
import heapq

def collections_example():
    stack = []
    stack.append(1)
    top = stack[-1]
    stack.pop()
    queue = deque([1])
    front = queue.popleft()
    heap = []
    heapq.heappush(heap, 3)
    smallest = heapq.heappop(heap)
    # 大根堆可压入 -x，弹出后取负。
    freq = Counter([3, 3, 4])
    graph = defaultdict(list)
    seen = {3, 4}
    return top, front, smallest, freq, graph, seen
# Python 标准库没有 TreeSet；有序列表插入是 O(n)，不可当作平衡树。
```

## 单调栈与单调队列

返回左侧最近严格较小元素的下标，以及长度 k 的滑动窗口最小值。均为 O(n)，要求 1 ≤ k ≤ n。

```python
from collections import deque

def previous_less(a):
    stack, res = [], []
    for i, x in enumerate(a):
        while stack and a[stack[-1]] >= x:
            stack.pop()
        res.append(stack[-1] if stack else -1)
        stack.append(i)
    return res

def window_min(a, k):
    if not 1 <= k <= len(a):
        raise ValueError('invalid window size')
    q, res = deque(), []
    for i, x in enumerate(a):
        while q and q[0] <= i - k:
            q.popleft()
        while q and a[q[-1]] >= x:
            q.pop()
        q.append(i)
        if i >= k - 1:
            res.append(a[q[0]])
    return res
```

## KMP 字符串匹配

返回所有匹配起点，支持重叠；空模式定义为匹配每个边界。O(n+m)。Java 下标按 UTF-16 char 计数。

```python
def kmp(text, pattern):
    m = len(pattern)
    if not m:
        return list(range(len(text) + 1))
    pi, j = [0] * m, 0
    for i in range(1, m):
        while j and pattern[i] != pattern[j]:
            j = pi[j - 1]
        if pattern[i] == pattern[j]:
            j += 1
        pi[i] = j
    res, j = [], 0
    for i, c in enumerate(text):
        while j and c != pattern[j]:
            j = pi[j - 1]
        if c == pattern[j]:
            j += 1
        if j == m:
            res.append(i - m + 1)
            j = pi[j - 1]
    return res
```

## Trie 字典树

支持插入和统计完整单词出现次数，O(单词长度)。Java 版本限定小写英文字母；Python 使用字典存边。

```python
class Trie:
    def __init__(self):
        self.children = [{}]
        self.counts = [0]

    def insert(self, s):
        p = 0
        for c in s:
            if c not in self.children[p]:
                self.children[p][c] = len(self.children)
                self.children.append({})
                self.counts.append(0)
            p = self.children[p][c]
        self.counts[p] += 1

    def count(self, s):
        p = 0
        for c in s:
            if c not in self.children[p]:
                return 0
            p = self.children[p][c]
        return self.counts[p]
```

## 并查集

路径压缩与按大小合并，均摊 O(α(n))。重复合并不改变集合大小。

```python
class DSU:
    def __init__(self, n):
        self.parent = list(range(n))
        self.size = [1] * n

    def find(self, x):
        while self.parent[x] != x:
            self.parent[x] = self.parent[self.parent[x]]
            x = self.parent[x]
        return x

    def union(self, a, b):
        a, b = self.find(a), self.find(b)
        if a == b:
            return False
        if self.size[a] < self.size[b]:
            a, b = b, a
        self.parent[b] = a
        self.size[a] += self.size[b]
        return True
```

## 可修改、可删除的编号堆

编号由调用方提供且当前唯一；支持插入、按编号修改、删除、取最小值，O(log n)。普通堆与标准库写法见前文。

```python
class IndexedHeap:
    def __init__(self):
        self.heap, self.pos = [], {}

    def _swap(self, i, j):
        h = self.heap
        h[i], h[j] = h[j], h[i]
        self.pos[h[i][1]], self.pos[h[j][1]] = i, j

    def _up(self, i):
        while i:
            p = (i - 1) // 2
            if self.heap[p][0] <= self.heap[i][0]:
                break
            self._swap(i, p)
            i = p
        return i

    def _down(self, i):
        h = self.heap
        while 2 * i + 1 < len(h):
            j = 2 * i + 1
            if j + 1 < len(h) and h[j + 1][0] < h[j][0]:
                j += 1
            if h[i][0] <= h[j][0]:
                break
            self._swap(i, j)
            i = j

    def insert(self, key, value):
        if key in self.pos:
            raise ValueError('duplicate id')
        self.pos[key] = len(self.heap)
        self.heap.append([value, key])
        self._up(len(self.heap) - 1)

    def change(self, key, value):
        i = self.pos[key]
        self.heap[i][0] = value
        self._down(self._up(i))

    def remove(self, key):
        i = self.pos[key]
        self._swap(i, len(self.heap) - 1)
        self.heap.pop()
        del self.pos[key]
        if i < len(self.heap):
            self._down(self._up(i))

    def peek(self):
        return tuple(self.heap[0])
```

## 哈希：开放寻址与链表法

整数集合，仅插入和查询。开放寻址表容量必须大于不同元素数；使用占用标记，不占用某个整数作空值。链表法允许键碰撞。

```python
class OpenHash:
    def __init__(self, capacity):
        self.keys = [None] * capacity

    def _locate(self, x):
        k = start = x % len(self.keys)
        while self.keys[k] is not None and self.keys[k] != x:
            k = (k + 1) % len(self.keys)
            if k == start:
                raise ValueError('table full')
        return k

    def add(self, x):
        self.keys[self._locate(x)] = x

    def contains(self, x):
        return self.keys[self._locate(x)] is not None

class ChainHash:
    def __init__(self, capacity):
        self.buckets = [[] for _ in range(capacity)]

    def add(self, x):
        bucket = self.buckets[x % len(self.buckets)]
        if x not in bucket:
            bucket.append(x)

    def contains(self, x):
        return x in self.buckets[x % len(self.buckets)]
```

## 字符串前缀哈希

查询 [l,r) 的哈希。固定 base=131、mod=1000000007；哈希相同可能碰撞，需要精确结果时再核对原字符串。O(n) 构建、O(1) 查询。Java 与 Python 此处均以 BMP 字符作为一致的对照输入。

```python
class StringHash:
    MOD, BASE = 1_000_000_007, 131

    def __init__(self, s):
        self.h, self.power = [0], [1]
        for c in s:
            self.h.append((self.h[-1] * self.BASE + ord(c) + 1) % self.MOD)
            self.power.append(self.power[-1] * self.BASE % self.MOD)

    def get(self, l, r):
        return (self.h[r] - self.h[l] * self.power[r - l]) % self.MOD
```

## 二叉树遍历

显式节点表示一般二叉树，不要求满二叉树。返回前序、中序、后序三种序列；迭代实现，O(n)。

```python
class TreeNode:
    def __init__(self, value, left=None, right=None):
        self.value, self.left, self.right = value, left, right

def traversals(root):
    pre, inorder, post = [], [], []
    stack = [root] if root else []
    while stack:
        u = stack.pop()
        pre.append(u.value)
        if u.right:
            stack.append(u.right)
        if u.left:
            stack.append(u.left)
    stack, cur = [], root
    while cur or stack:
        while cur:
            stack.append(cur)
            cur = cur.left
        cur = stack.pop()
        inorder.append(cur.value)
        cur = cur.right
    stack = [root] if root else []
    while stack:
        u = stack.pop()
        post.append(u.value)
        if u.left:
            stack.append(u.left)
        if u.right:
            stack.append(u.right)
    return pre, inorder, post[::-1]
```

## 哈夫曼树与合并代价

非负权值；每次合并最小两项，返回带权路径长度。空集合或单节点代价为 0，O(n log n)。贪心中的合并果子使用同一算法。

```python
import heapq

def huffman_cost(weights):
    q, answer = list(weights), 0
    heapq.heapify(q)
    while len(q) > 1:
        total = heapq.heappop(q) + heapq.heappop(q)
        answer += total
        heapq.heappush(q, total)
    return answer
```

## 树状数组：点加、区间加与区间和

本节使用一基下标。prefix(r) 查询 [1,r]；rangeAdd(l,r,v) 更新闭区间 [l,r]。单次操作 O(log n)，构造后为零数组。

```python
class Fenwick:
    def __init__(self, n):
        self.n, self.tree = n, [0] * (n + 1)

    def add(self, i, v):
        if i <= 0:
            raise ValueError('indices start at 1')
        while i <= self.n:
            self.tree[i] += v
            i += i & -i

    def prefix(self, i):
        total = 0
        while i > 0:
            total += self.tree[i]
            i -= i & -i
        return total

    def range_sum(self, l, r):
        return self.prefix(r) - self.prefix(l - 1)

class RangeFenwick:
    def __init__(self, n):
        self.b1, self.b2 = Fenwick(n), Fenwick(n)

    def range_add(self, l, r, v):
        self.b1.add(l, v)
        self.b1.add(r + 1, -v)
        self.b2.add(l, v * (l - 1))
        self.b2.add(r + 1, -v * r)

    def prefix(self, r):
        return self.b1.prefix(r) * r - self.b2.prefix(r)

    def range_sum(self, l, r):
        return self.prefix(r) - self.prefix(l - 1)

    def point(self, i):
        return self.range_sum(i, i)
```

## 线段树：懒标记区间加、区间和

零基闭区间 [l,r]，要求原数组非空；更新、查询 O(log n)。

```python
class SegmentTree:
    def __init__(self, a):
        if not a:
            raise ValueError('nonempty array required')
        self.n = len(a)
        self.total, self.lazy = [0] * (4 * self.n), [0] * (4 * self.n)
        self._build(1, 0, self.n - 1, a)

    def _build(self, u, l, r, a):
        if l == r:
            self.total[u] = a[l]
            return
        m = (l + r) // 2
        self._build(u * 2, l, m, a)
        self._build(u * 2 + 1, m + 1, r, a)
        self._pull(u)

    def _pull(self, u):
        self.total[u] = self.total[u * 2] + self.total[u * 2 + 1]

    def _apply(self, u, l, r, v):
        self.total[u] += (r - l + 1) * v
        self.lazy[u] += v

    def _push(self, u, l, r):
        m = (l + r) // 2
        self._apply(u * 2, l, m, self.lazy[u])
        self._apply(u * 2 + 1, m + 1, r, self.lazy[u])
        self.lazy[u] = 0

    def add(self, ql, qr, v):
        def update(u, l, r):
            if ql <= l and r <= qr:
                self._apply(u, l, r, v)
                return
            self._push(u, l, r)
            m = (l + r) // 2
            if ql <= m:
                update(u * 2, l, m)
            if qr > m:
                update(u * 2 + 1, m + 1, r)
            self._pull(u)
        update(1, 0, self.n - 1)

    def query(self, ql, qr):
        def get(u, l, r):
            if ql <= l and r <= qr:
                return self.total[u]
            self._push(u, l, r)
            m, res = (l + r) // 2, 0
            if ql <= m:
                res += get(u * 2, l, m)
            if qr > m:
                res += get(u * 2 + 1, m + 1, r)
            return res
        return get(1, 0, self.n - 1)
```

## 树形 DP：最大权独立集

输入非空无向树与节点权值；相邻节点不能同时选，可全不选。迭代建树、逆序转移，O(n)。

```python
def tree_independent_set(g, weight):
    n, order, parent = len(g), [0], [-2] * len(g)
    parent[0] = -1
    for u in order:
        for v in g[u]:
            if parent[v] == -2:
                parent[v] = u
                order.append(v)
    dp = [[0, 0] for _ in range(n)]
    for u in reversed(order):
        dp[u][1] += weight[u]
        p = parent[u]
        if p >= 0:
            dp[p][0] += max(dp[u])
            dp[p][1] += dp[u][0]
    return max(dp[0])
```

# 搜索与图论

## DFS：全排列与 N 皇后

返回全部解，内存随解数量增长；递归仅用于搜索深度较小的题。全排列 O(n·n!)；N 皇后按列与对角线剪枝。

```python
def permutations(n):
    ans, path, used = [], [], [False] * n
    def dfs():
        if len(path) == n:
            ans.append(path[:])
            return
        for x in range(n):
            if not used[x]:
                used[x] = True
                path.append(x)
                dfs()
                path.pop()
                used[x] = False
    dfs()
    return ans

def n_queens(n):
    ans, path, col, d1, d2 = [], [], set(), set(), set()
    def dfs(r):
        if r == n:
            ans.append(path[:])
            return
        for c in range(n):
            if c not in col and r + c not in d1 and r - c not in d2:
                col.add(c); d1.add(r + c); d2.add(r - c); path.append(c)
                dfs(r + 1)
                path.pop(); col.remove(c); d1.remove(r + c); d2.remove(r - c)
    dfs(0)
    return ans
```

## 树的重心

对应原模板“树的中心”的实际算法：返回删除单个节点后最大连通块大小的最小值；输入非空无向树。O(n)。

```python
def centroid_balance(g):
    n, parent, order = len(g), [-2] * len(g), [0]
    parent[0] = -1
    for u in order:
        for v in g[u]:
            if parent[v] == -2:
                parent[v] = u
                order.append(v)
    size, best = [1] * n, n
    for u in reversed(order):
        largest = 0
        for v in g[u]:
            if parent[v] == u:
                size[u] += size[v]
                largest = max(largest, size[v])
        best = min(best, max(largest, n - size[u]))
    return best
```

## BFS：图的层次与迷宫

无权图，返回源点到所有点的距离，不可达为 -1。迷宫用 0 表示可走、1 表示障碍，起终点分别为左上角和右下角。O(V+E) / O(nm)。

```python
from collections import deque

def bfs(g, source):
    d, q = [-1] * len(g), deque([source])
    d[source] = 0
    while q:
        u = q.popleft()
        for v in g[u]:
            if d[v] == -1:
                d[v] = d[u] + 1
                q.append(v)
    return d

def maze_distance(a):
    if not a or not a[0] or a[0][0] or a[-1][-1]:
        return -1
    n, m = len(a), len(a[0])
    d, q = [[-1] * m for _ in range(n)], deque([(0, 0)])
    d[0][0] = 0
    while q:
        x, y = q.popleft()
        for dx, dy in ((-1, 0), (0, 1), (1, 0), (0, -1)):
            nx, ny = x + dx, y + dy
            if 0 <= nx < n and 0 <= ny < m and a[nx][ny] == 0 and d[nx][ny] == -1:
                d[nx][ny] = d[x][y] + 1
                q.append((nx, ny))
    return d[-1][-1]
```

## Dijkstra：朴素与堆优化

边权非负。朴素版矩阵无边设为 INF，O(n²)；堆版邻接项为 {终点,权重}，O((V+E) log V)。不可达为 Java Long.MAX_VALUE/4、Python inf。

```python
import heapq
from math import inf

def dijkstra_dense(g, source):
    n, d, used = len(g), [inf] * len(g), [False] * len(g)
    d[source] = 0
    for _ in range(n):
        u = min((v for v in range(n) if not used[v]), key=lambda v: d[v], default=-1)
        if u == -1 or d[u] == inf:
            break
        used[u] = True
        for v, w in enumerate(g[u]):
            d[v] = min(d[v], d[u] + w)
    return d

def dijkstra(g, source):
    d, q = [inf] * len(g), [(0, source)]
    d[source] = 0
    while q:
        dist, u = heapq.heappop(q)
        if dist != d[u]:
            continue
        for v, w in g[u]:
            if d[v] > dist + w:
                d[v] = dist + w
                heapq.heappush(q, (d[v], v))
    return d
```

## Bellman-Ford：最多 k 条边

边表每项为 {起点,终点,权重}。每轮读取上一轮副本，允许负权与负环，但限制最多 k 条边；不可达为 INF / inf。O(kE)。

```python
from math import inf

def bellman_ford(n, edges, source, k):
    d = [inf] * n
    d[source] = 0
    for _ in range(k):
        old = d[:]
        for u, v, w in edges:
            if old[u] != inf:
                d[v] = min(d[v], old[u] + w)
    return d
```

## SPFA

允许负权；检测到从源点可达的负环时抛出异常。邻接项为 {终点,权重}。最坏 O(VE)，不可达为 INF / inf。

```python
from collections import deque
from math import inf

def spfa(g, source):
    n = len(g)
    d, count, queued = [inf] * n, [0] * n, [False] * n
    d[source], queued[source] = 0, True
    q = deque([source])
    while q:
        u = q.popleft()
        queued[u] = False
        for v, w in g[u]:
            if d[v] > d[u] + w:
                d[v] = d[u] + w
                count[v] = count[u] + 1
                if count[v] >= n:
                    raise ValueError('reachable negative cycle')
                if not queued[v]:
                    queued[v] = True
                    q.append(v)
    return d
```

## Floyd 全源最短路

允许负权，要求没有负环；矩阵对角线为 0，无边为 INF / inf，重边建图取最小值。返回副本，O(n³)。

```python
from math import inf

def floyd(g):
    d = [row[:] for row in g]
    for k in range(len(d)):
        for i in range(len(d)):
            if d[i][k] == inf:
                continue
            for j in range(len(d)):
                if d[k][j] != inf:
                    d[i][j] = min(d[i][j], d[i][k] + d[k][j])
    return d
```

## 最小生成树：Prim 与 Kruskal

无向图，允许负权与重边。Prim 读矩阵，无边为 INF / inf，O(n²)；Kruskal 读 {u,v,w} 边表，依赖前文 DSU，O(E log E)。非连通返回 Java null、Python None，空图返回 0。

```python
from math import inf

def prim(g):
    n = len(g)
    if not n:
        return 0
    d, used, answer = [inf] * n, [False] * n, 0
    d[0] = 0
    for _ in range(n):
        u = min((v for v in range(n) if not used[v]), key=lambda v: d[v])
        if d[u] == inf:
            return None
        answer += d[u]
        used[u] = True
        for v in range(n):
            if not used[v]:
                d[v] = min(d[v], g[u][v])
    return answer

def kruskal(n, edges):
    dsu, answer, count = DSU(n), 0, 0
    for u, v, w in sorted(edges, key=lambda e: e[2]):
        if dsu.union(u, v):
            answer += w
            count += 1
    return answer if n == 0 or count == n - 1 else None
```

## 二分图：染色与匈牙利算法

染色读取无向邻接表，O(V+E)。匹配输入仅为左侧点的邻接表，邻接值是右侧编号，O(VE)；使用迭代增广，避免长交替路的递归栈限制。

```python
from collections import deque

def is_bipartite(g):
    color = [0] * len(g)
    for s in range(len(g)):
        if color[s]:
            continue
        color[s], q = 1, deque([s])
        while q:
            u = q.popleft()
            for v in g[u]:
                if color[v] == color[u]:
                    return False
                if not color[v]:
                    color[v] = 3 - color[u]
                    q.append(v)
    return True

def max_matching(g, right_count):
    left, right, answer = [-1] * len(g), [-1] * right_count, 0
    for s in range(len(g)):
        prev_right, seen = [-1] * right_count, {s}
        q, end = deque([s]), -1
        while q and end == -1:
            u = q.popleft()
            for v in g[u]:
                if prev_right[v] != -1:
                    continue
                prev_right[v] = u
                if right[v] == -1:
                    end = v
                    break
                if right[v] not in seen:
                    seen.add(right[v])
                    q.append(right[v])
        if end != -1:
            answer += 1
            while end != -1:
                u = prev_right[end]
                old = left[u]
                left[u], right[end], end = end, u, old
    return answer
```

## 拓扑排序

有向图；存在环返回 null / None，否则返回一个拓扑序，O(V+E)。

```python
from collections import deque

def topological_sort(g):
    indegree = [0] * len(g)
    for row in g:
        for v in row:
            indegree[v] += 1
    q, order = deque(i for i, d in enumerate(indegree) if d == 0), []
    while q:
        u = q.popleft()
        order.append(u)
        for v in g[u]:
            indegree[v] -= 1
            if indegree[v] == 0:
                q.append(v)
    return order if len(order) == len(g) else None
```

# 数学知识

## 质数：试除、分解与筛法

factorize 输入正整数，返回质因子到指数的映射；筛法统一包含上界 n。试除 O(√n)，埃氏筛 O(n log log n)，线性筛 O(n)。

```python
def is_prime(n):
    if n < 2:
        return False
    p = 2
    while p * p <= n:
        if n % p == 0:
            return False
        p += 1
    return True

def factorize(n):
    if n < 1:
        raise ValueError('positive integer required')
    factors, p = {}, 2
    while p * p <= n:
        if n % p == 0:
            count = 0
            while n % p == 0:
                n //= p
                count += 1
            factors[p] = count
        p += 1
    if n > 1:
        factors[n] = 1
    return factors

def sieve(n):
    composite, primes = [False] * (n + 1), []
    for i in range(2, n + 1):
        if not composite[i]:
            primes.append(i)
            for j in range(i * i, n + 1, i):
                composite[j] = True
    return primes

def linear_sieve(n):
    composite, primes = [False] * (n + 1), []
    for i in range(2, n + 1):
        if not composite[i]:
            primes.append(i)
        for p in primes:
            if i * p > n:
                break
            composite[i * p] = True
            if i % p == 0:
                break
    return primes
```

## 约数、约数个数与约数之和

divisors 输入正整数，O(√n) 枚举后排序。个数与和从质因子指数表计算，依赖前文 factorize；模数由调用方传入。Java 要求 1 ≤ mod ≤ 2147483647，使中间模乘可用 long。

```python
def divisors(n):
    if n < 1:
        raise ValueError('positive integer required')
    ans, i = [], 1
    while i * i <= n:
        if n % i == 0:
            ans.append(i)
            if i != n // i:
                ans.append(n // i)
        i += 1
    return sorted(ans)

def divisor_count(factors, mod):
    ans = 1 % mod
    for e in factors.values():
        ans = ans * (e + 1) % mod
    return ans

def divisor_sum(factors, mod):
    ans = 1 % mod
    for p, e in factors.items():
        term = 1 % mod
        for _ in range(e):
            term = (term * p + 1) % mod
        ans = ans * term % mod
    return ans
```

## 欧几里得、扩展欧几里得与线性同余方程

输入非负整数且不同时为 0；扩展算法返回 (g,x,y)，满足 ax+by=g。解 ax≡b (mod m) 时要求 m>0，无解返回 null / None。Java 的中间系数乘法须在 long 范围内。

```python
def gcd(a, b):
    while b:
        a, b = b, a % b
    return a

def exgcd(a, b):
    if b == 0:
        return a, 1, 0
    g, x, y = exgcd(b, a % b)
    return g, y, x - (a // b) * y

def solve_congruence(a, b, m):
    g, x, _ = exgcd(a, m)
    if b % g:
        return None
    return x * (b // g) % (m // g)
```

## 快速幂

b≥0，mod>0；O(log b)。Java 限定 mod≤2147483647，避免 long 模乘溢出。mod=1 时正确返回 0。

```python
def mod_pow(a, b, mod):
    if b < 0 or mod <= 0:
        raise ValueError('b >= 0 and mod > 0 required')
    result, a = 1 % mod, a % mod
    while b:
        if b & 1:
            result = result * a % mod
        a = a * a % mod
        b >>= 1
    return result
```

## 高斯消元

n 个方程、n 个未知数，增广矩阵 n×(n+1)，直接修改矩阵。返回 0=唯一解、1=无穷多解、2=无解；唯一解存最后一列。O(n³)，按 eps=1e-9 判断零。

```python
def gauss(a, eps=1e-9):
    n, r = len(a), 0
    for c in range(n):
        t = max(range(r, n), key=lambda i: abs(a[i][c]))
        if abs(a[t][c]) < eps:
            continue
        a[r], a[t] = a[t], a[r]
        for j in range(n, c - 1, -1):
            a[r][j] /= a[r][c]
        for i in range(r + 1, n):
            if abs(a[i][c]) > eps:
                for j in range(n, c - 1, -1):
                    a[i][j] -= a[r][j] * a[i][c]
        r += 1
    if r < n:
        return 2 if any(abs(a[i][n]) > eps for i in range(r, n)) else 1
    for i in range(n - 1, -1, -1):
        for j in range(i + 1, n):
            a[i][n] -= a[i][j] * a[j][n]
    return 0
```

## 组合数：递推、阶乘逆元、Lucas 与精确值

递推适用任意正模数。阶乘逆元要求 p 为质数且 n<p；Lucas 要求 p 为质数，单次小组合 O(p)，适合较小 p。Java p≤2147483647，依赖前文 modPow；精确组合数使用 BigInteger / 整数。

```python
from math import comb

def combination_table(n, mod):
    c = [[0] * (n + 1) for _ in range(n + 1)]
    for i in range(n + 1):
        c[i][0] = 1 % mod
        for j in range(1, i + 1):
            c[i][j] = (c[i - 1][j - 1] + c[i - 1][j]) % mod
    return c

class CombPrime:
    def __init__(self, n, p):
        if not 0 <= n < p or p < 2:
            raise ValueError('0 <= n < prime p required')
        self.p, self.fact, self.inv_fact = p, [1] * (n + 1), [1] * (n + 1)
        for i in range(1, n + 1):
            self.fact[i] = self.fact[i - 1] * i % p
        self.inv_fact[n] = pow(self.fact[n], p - 2, p)
        for i in range(n, 0, -1):
            self.inv_fact[i - 1] = self.inv_fact[i] * i % p

    def choose(self, n, k):
        if k < 0 or k > n:
            return 0
        return self.fact[n] * self.inv_fact[k] * self.inv_fact[n - k] % self.p

def lucas(n, k, p):
    if p < 2:
        raise ValueError('prime p >= 2 required')
    def small(n, k):
        if k > n:
            return 0
        a, b = 1, 1
        for i in range(1, min(k, n - k) + 1):
            a = a * (n - i + 1) % p
            b = b * i % p
        return a * pow(b, p - 2, p) % p
    ans = 1
    while n or k:
        ans = ans * small(n % p, k % p) % p
        n //= p
        k //= p
    return ans

def exact_comb(n, k):
    return comb(n, k) if 0 <= k <= n else 0
```

## 卡特兰数

n≥0，返回精确值 C(2n,n)/(n+1)，依赖 exactComb / exact_comb；模任意正数可在得到精确值后取模。

```python
def catalan(n):
    return exact_comb(2 * n, n) // (n + 1)
```

# 贪心

## 合并果子、区间覆盖、分组与选点

合并果子调用前文 huffmanCost / huffman_cost。其余均为闭区间，O(n log n)。区间覆盖要求 start<end，无法覆盖返回 -1；分组中端点相接算冲突，选点中一个端点可以覆盖多个区间。

```python
import heapq
from math import inf

def cover_intervals(start, end, ranges):
    a, i, count = sorted(ranges), 0, 0
    while start < end:
        next_end = start
        while i < len(a) and a[i][0] <= start:
            next_end = max(next_end, a[i][1])
            i += 1
        if next_end == start:
            return -1
        start, count = next_end, count + 1
    return count

def group_intervals(ranges):
    ends = []
    for l, r in sorted(ranges):
        if ends and ends[0] < l:
            heapq.heappop(ends)
        heapq.heappush(ends, r)
    return len(ends)

def select_points(ranges):
    point, count = -inf, 0
    for l, r in sorted(ranges, key=lambda x: x[1]):
        if point < l:
            point, count = r, count + 1
    return count
```

# 动态规划

## 背包：01、完全、多重与分组

容量至多为 m，可不选物品。物品体积为正整数，价值允许负数。01 / 完全背包 O(nm)，多重背包二进制拆分后 O(mΣlog(s+1))，分组背包 O(mΣ组大小)。Java 价值与状态用 long。

```python
def knapsack_01(m, items):
    f = [0] * (m + 1)
    for v, w in items:
        for j in range(m, v - 1, -1):
            f[j] = max(f[j], f[j - v] + w)
    return f[m]

def knapsack_complete(m, items):
    f = [0] * (m + 1)
    for v, w in items:
        for j in range(v, m + 1):
            f[j] = max(f[j], f[j - v] + w)
    return f[m]

def knapsack_bounded(m, items):
    f = [0] * (m + 1)
    for v, w, count in items:
        remaining, k = min(count, m // v), 1
        while remaining:
            take = min(k, remaining)
            remaining -= take
            volume, value = v * take, w * take
            for j in range(m, volume - 1, -1):
                f[j] = max(f[j], f[j - volume] + value)
            k *= 2
    return f[m]

def knapsack_grouped(m, groups):
    f = [0] * (m + 1)
    for group in groups:
        old = f[:]
        for j in range(m + 1):
            for v, w in group:
                if j >= v:
                    f[j] = max(f[j], old[j - v] + w)
    return f[m]
```

## 线性 DP：数字三角形

第 i 行恰有 i+1 项；求从顶到底的最大路径和，允许负数。自底向上，O(n²) 时间、O(n) 额外空间；空三角形返回 0。

```python
def triangle_max(a):
    if not a:
        return 0
    f = a[-1][:]
    for i in range(len(a) - 2, -1, -1):
        for j in range(i + 1):
            f[j] = a[i][j] + max(f[j], f[j + 1])
    return f[0]
```

## 最长上升子序列

严格上升；lis 返回一条正向序列，O(n²)，lisLength 返回长度，O(n log n)。空数组返回空序列或 0。

```python
from bisect import bisect_left

def lis(a):
    if not a:
        return []
    f, prev, best = [1] * len(a), [-1] * len(a), 0
    for i, x in enumerate(a):
        for j in range(i):
            if a[j] < x and f[j] + 1 > f[i]:
                f[i], prev[i] = f[j] + 1, j
        if f[i] > f[best]:
            best = i
    ans = []
    while best != -1:
        ans.append(a[best])
        best = prev[best]
    return ans[::-1]

def lis_length(a):
    tails = []
    for x in a:
        i = bisect_left(tails, x)
        if i == len(tails):
            tails.append(x)
        else:
            tails[i] = x
    return len(tails)
```

## 最长公共子序列

返回长度，O(nm) 时间、O(m) 空间；Java 按 UTF-16 char 比较。

```python
def lcs_length(a, b):
    f = [0] * (len(b) + 1)
    for x in a:
        diagonal = 0
        for j, y in enumerate(b, 1):
            old = f[j]
            f[j] = diagonal + 1 if x == y else max(f[j], f[j - 1])
            diagonal = old
    return f[-1]
```

## 区间 DP：石子合并

每次只能合并相邻两堆，求最小总代价，线性排列；O(n³) 时间、O(n²) 空间。

```python
from math import inf

def stone_merge(a):
    n = len(a)
    if not n:
        return 0
    s = [0]
    for x in a:
        s.append(s[-1] + x)
    f = [[0] * n for _ in range(n)]
    for length in range(2, n + 1):
        for l in range(n - length + 1):
            r = l + length - 1
            f[l][r] = min(f[l][k] + f[k + 1][r] for k in range(l, r)) + s[r + 1] - s[l]
    return f[0][-1]
```

# 高精度

## 大整数：加、减、乘、除

输入为十进制非负整数字符串；减法允许负结果，除数必须大于 0，返回商和余数。Java 使用 BigInteger，Python 使用整数；除法不转浮点数。

```python
def big_add(a, b):
    return str(int(a) + int(b))

def big_subtract(a, b):
    return str(int(a) - int(b))

def big_multiply(a, b):
    return str(int(a) * int(b))

def big_divide(a, b):
    divisor = int(b)
    if divisor <= 0:
        raise ValueError('positive divisor required')
    q, r = divmod(int(a), divisor)
    return str(q), str(r)
# 极长十进制字符串可能触及 Python 的整数转字符串位数限制。
```

## 手写高精度：十进制数位数组

与 C++ 原模板的思路对应。数位低位在前，至少一位，输入须规范化（除零本身外无高位零）。实现加法、非负减法、乘非负小整数、除正小整数；减法要求 A≥B。Java 小整数 ≤10^9，中间进位用 long。

```python
def digits(s):
    a = [int(c) for c in reversed(s)]
    _trim_digits(a)
    return a

def _trim_digits(a):
    while len(a) > 1 and a[-1] == 0:
        a.pop()

def add_digits(a, b):
    c, carry, i = [], 0, 0
    while i < max(len(a), len(b)) or carry:
        carry += (a[i] if i < len(a) else 0) + (b[i] if i < len(b) else 0)
        c.append(carry % 10)
        carry //= 10
        i += 1
    return c

def subtract_digits(a, b):
    c, borrow = [], 0
    for i, x in enumerate(a):
        x -= borrow + (b[i] if i < len(b) else 0)
        c.append(x % 10)
        borrow = int(x < 0)
    _trim_digits(c)
    return c

def multiply_digits(a, b):
    c, carry, i = [], 0, 0
    while i < len(a) or carry:
        if i < len(a):
            carry += a[i] * b
        c.append(carry % 10)
        carry //= 10
        i += 1
    _trim_digits(c)
    return c

def divide_digits(a, b):
    c, r = [], 0
    for x in reversed(a):
        r = r * 10 + x
        c.append(r // b)
        r %= b
    c.reverse()
    _trim_digits(c)
    return c, r
```

# 莫队

## 普通莫队：区间不同元素数

零基闭区间 [l,r]，支持任意整数值；返回原查询顺序的答案。左右端点移动时维护频率，右端点采用奇偶块排序。典型 n 与查询数同阶时 O((n+q)√n)。

```python
from math import isqrt

def mo_distinct(a, queries):
    block = max(1, isqrt(len(a)))
    order = sorted(range(len(queries)), key=lambda i: (
        queries[i][0] // block,
        queries[i][1] if (queries[i][0] // block) % 2 == 0 else -queries[i][1]))
    freq, l, r, ans = {}, 0, -1, [0] * len(queries)
    def adjust(x, delta):
        freq[x] = freq.get(x, 0) + delta
        if not freq[x]:
            del freq[x]
    for i in order:
        ql, qr = queries[i]
        while l > ql:
            l -= 1
            adjust(a[l], 1)
        while r < qr:
            r += 1
            adjust(a[r], 1)
        while l < ql:
            adjust(a[l], -1)
            l += 1
        while r > qr:
            adjust(a[r], -1)
            r -= 1
        ans[i] = len(freq)
    return ans
```

## 带修莫队：区间不同元素数

操作为 {0,l,r}=查询、{1,index,value}=单点赋值；零基闭区间，返回查询出现顺序的答案。时间指针通过交换旧值和新值实现前进、回滚。n、操作数同阶时典型 O(n^(5/3))，块长取 n^(2/3)；Java 依赖上一节 moAdjust。

```python
def mo_distinct_with_updates(initial, operations):
    a, changes, queries = list(initial), [], []
    for kind, x, y in operations:
        if kind == 0:
            queries.append((x, y, len(changes), len(queries)))
        else:
            changes.append([x, y])
    block = max(1, int(max(1, len(a)) ** (2 / 3)))
    queries.sort(key=lambda q: (q[0] // block, q[1] // block, q[2]))
    freq, l, r, time, ans = {}, 0, -1, 0, [0] * len(queries)
    def adjust(x, delta):
        freq[x] = freq.get(x, 0) + delta
        if not freq[x]:
            del freq[x]
    def change(t):
        p, value = changes[t]
        if l <= p <= r:
            adjust(a[p], -1)
            adjust(value, 1)
        a[p], changes[t][1] = value, a[p]
    for ql, qr, qt, i in queries:
        while time < qt:
            change(time)
            time += 1
        while time > qt:
            time -= 1
            change(time)
        while l > ql:
            l -= 1
            adjust(a[l], 1)
        while r < qr:
            r += 1
            adjust(a[r], 1)
        while l < ql:
            adjust(a[l], -1)
            l += 1
        while r > qr:
            adjust(a[r], -1)
            r -= 1
        ans[i] = len(freq)
    return ans
```

