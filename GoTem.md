# Go 算法模板

与 [C++](./CppTem.md)、[Java](./JavaTem.md)、[Python](./PythonTem.md) 按知识点对应的 49 个算法小节。导航：[README](./README.md) · [C++](./CppTem.md) · [Java](./JavaTem.md) · [Python](./PythonTem.md) · [Go](./GoTem.md) · [TypeScript](./TypeScriptTem.md) · [Rust](./RustTem.md)。

## 使用约定

- 默认数组、图节点从 **0** 开始；树状数组从 **1** 开始。区间开闭、适用条件与依赖见各节。只复制所需实现与依赖，再补题目输入输出。
- 输入须满足有效下标与各节前提；浮点高斯消元使用容差。哈希比较可能碰撞。
- 使用 Go 1.21 及以上（含内置 `min`、`max`），面向 64 位平台。索引使用 `int`，数值和、距离使用 `int64`；中间结果也须在范围内。
- 各节代码放进 `package main` 文件。按实际选用代码导入以下标准库（Go 不允许未使用的导入），并补 `func main()`：

```go
package main

import (
	"container/heap"
	"io"
	"math"
	"math/big"
	"os"
	"sort"
	"strconv"
	"strings"
)

func main() {
	// 放入所需调用；函数和类型定义放在 main 外。
}
```

## 内容导航

[基础算法](#基础算法) · [数据结构](#数据结构) · [搜索与图论](#搜索与图论) · [数学知识](#数学知识) · [贪心](#贪心) · [动态规划](#动态规划) · [高精度](#高精度) · [莫队](#莫队)

# 基础算法

## 快速输入

批量读取整数。Go 返回读取或解析错误；TypeScript 的解析函数接收输入字符串；Rust 读取标准输入。输入含 ASCII 空白，整数须在对应类型范围内。

```go
func ReadInts() ([]int64, error) {
	data, err := io.ReadAll(os.Stdin)
	if err != nil {
		return nil, err
	}
	fields := strings.Fields(string(data))
	a := make([]int64, len(fields))
	for i, s := range fields {
		a[i], err = strconv.ParseInt(s, 10, 64)
		if err != nil {
			return nil, err
		}
	}
	return a, nil
}
```

## 排序：快速排序与归并排序

快速排序原地修改，最坏 O(n²)；归并排序 O(n log n)。Rust 切片版空数组安全。

```go
func QuickSort(a []int64) {
	if len(a) < 2 {
		return
	}
	i, j, x := -1, len(a), a[(len(a)-1)/2]
	for i < j {
		i++
		for a[i] < x {
			i++
		}
		j--
		for a[j] > x {
			j--
		}
		if i < j {
			a[i], a[j] = a[j], a[i]
		}
	}
	QuickSort(a[:j+1])
	QuickSort(a[j+1:])
}
func MergeSort(a []int64) {
	if len(a) < 2 {
		return
	}
	m := len(a) / 2
	MergeSort(a[:m])
	MergeSort(a[m:])
	tmp := make([]int64, 0, len(a))
	i, j := 0, m
	for i < m && j < len(a) {
		if a[i] <= a[j] {
			tmp = append(tmp, a[i])
			i++
		} else {
			tmp = append(tmp, a[j])
			j++
		}
	}
	tmp = append(tmp, a[i:m]...)
	tmp = append(tmp, a[j:]...)
	copy(a, tmp)
}
```

## 前缀和：一维与二维

一维查询 [l,r)，二维查询 [x1,x2)×[y1,y2)。O(n)/O(nm) 构建，O(1) 查询。

```go
func Prefix(a []int64) []int64 {
	s := make([]int64, len(a)+1)
	for i, x := range a {
		s[i+1] = s[i] + x
	}
	return s
}
func Prefix2D(a [][]int64) [][]int64 {
	n, m := len(a), 0
	if n > 0 {
		m = len(a[0])
	}
	s := make([][]int64, n+1)
	for i := range s {
		s[i] = make([]int64, m+1)
	}
	for i := 0; i < n; i++ {
		for j := 0; j < m; j++ {
			s[i+1][j+1] = a[i][j] + s[i][j+1] + s[i+1][j] - s[i][j]
		}
	}
	return s
}
func RectSum(s [][]int64, x1, y1, x2, y2 int) int64 {
	return s[x2][y2] - s[x1][y2] - s[x2][y1] + s[x1][y1]
}
```

## 差分：一维与二维

在零数组上区间加，更新左闭右开；差分数组比原数组多一位，二维多一行一列。

```go
func RangeAdd(d []int64, l, r int, v int64) { d[l] += v; d[r] -= v }
func Restore(d []int64) []int64 {
	a := make([]int64, len(d)-1)
	var s int64
	for i := range a {
		s += d[i]
		a[i] = s
	}
	return a
}
func RectAdd(d [][]int64, x1, y1, x2, y2 int, v int64) {
	d[x1][y1] += v
	d[x2][y1] -= v
	d[x1][y2] -= v
	d[x2][y2] += v
}
func Restore2D(d [][]int64) [][]int64 {
	n, m := len(d)-1, len(d[0])-1
	a := make([][]int64, n)
	for i := range a {
		a[i] = make([]int64, m)
		for j := range a[i] {
			a[i][j] = d[i][j]
			if i > 0 {
				a[i][j] += a[i-1][j]
			}
			if j > 0 {
				a[i][j] += a[i][j-1]
			}
			if i > 0 && j > 0 {
				a[i][j] -= a[i-1][j-1]
			}
		}
	}
	return a
}
```

## 二分：整数边界与浮点数

有序数组第一个 ≥x / >x 的位置，不存在返回 n；浮点二分求有限非负数平方根。

```go
func LowerBound(a []int64, x int64) int {
	return sort.Search(len(a), func(i int) bool { return a[i] >= x })
}
func UpperBound(a []int64, x int64) int {
	return sort.Search(len(a), func(i int) bool { return a[i] > x })
}
func SqrtBinary(x float64) float64 {
	if x < 0 || math.IsNaN(x) || math.IsInf(x, 0) {
		panic("invalid x")
	}
	l, r := 0.0, math.Max(1, x)
	for i := 0; i < 1100; i++ {
		m := l + (r-l)/2
		if m == 0 || m <= x/m {
			l = m
		} else {
			r = m
		}
	}
	return l
}
```

## 位运算

lowbit 输入正整数，popcount 统计非负整数二进制 1 的数量。TypeScript 使用 bigint，避免位运算隐式截断为 32 位。

```go
func Lowbit(x uint64) uint64 { return x & -x }
func Popcount(x uint64) int {
	count := 0
	for x != 0 {
		x &= x - 1
		count++
	}
	return count
}
```

## 区间合并

闭区间合并，端点相接视为重叠。O(n log n)，不修改原数组。

```go
func MergeIntervals(ranges [][2]int64) [][2]int64 {
	a := append([][2]int64(nil), ranges...)
	sort.Slice(a, func(i, j int) bool { return a[i][0] < a[j][0] })
	res := [][2]int64{}
	for _, x := range a {
		if len(res) == 0 || res[len(res)-1][1] < x[0] {
			res = append(res, x)
		} else if res[len(res)-1][1] < x[1] {
			res[len(res)-1][1] = x[1]
		}
	}
	return res
}
```

## 离散化

返回排序去重后的值与原数组的零基排名，O(n log n)。

```go
func Compress(a []int64) ([]int64, []int) {
	values := append([]int64(nil), a...)
	sort.Slice(values, func(i, j int) bool { return values[i] < values[j] })
	u := values[:0]
	for _, x := range values {
		if len(u) == 0 || u[len(u)-1] != x {
			u = append(u, x)
		}
	}
	rank := make([]int, len(a))
	for i, x := range a {
		rank[i] = sort.Search(len(u), func(j int) bool { return u[j] >= x })
	}
	return u, rank
}
```

# 数据结构

## 数组模拟单链表与双链表

数组模拟链表。0 为单链表头哨兵；双链表用 0、1 为左右哨兵。插入返回稳定编号，删除不复用编号。O(1) 更新，删除须针对有效节点。

```go
type SingleList struct {
	Value []int64
	Next  []int
}

func NewSingleList() *SingleList { return &SingleList{[]int64{0}, []int{-1}} }
func (s *SingleList) InsertAfter(k int, x int64) int {
	i := len(s.Value)
	s.Value = append(s.Value, x)
	s.Next = append(s.Next, s.Next[k])
	s.Next[k] = i
	return i
}
func (s *SingleList) RemoveAfter(k int) {
	if s.Next[k] != -1 {
		s.Next[k] = s.Next[s.Next[k]]
	}
}

type DoubleList struct {
	Value       []int64
	Left, Right []int
}

func NewDoubleList() *DoubleList { return &DoubleList{[]int64{0, 0}, []int{-1, 0}, []int{1, -1}} }
func (s *DoubleList) InsertAfter(k int, x int64) int {
	i := len(s.Value)
	s.Value = append(s.Value, x)
	s.Left = append(s.Left, k)
	s.Right = append(s.Right, s.Right[k])
	s.Left[s.Right[k]] = i
	s.Right[k] = i
	return i
}
func (s *DoubleList) Remove(k int) { s.Right[s.Left[k]] = s.Right[k]; s.Left[s.Right[k]] = s.Left[k] }
```

## 栈、队列、堆与标准集合

栈、队列与最小堆。队列避免从动态数组头部逐项删除；最小堆用于后文 Dijkstra、哈夫曼与区间分组。Go 的 PairHeap 使用标准 container/heap；TypeScript 的 Heap 为可传比较器的通用堆。

```go
type PairHeap [][2]int64              // {优先级, 附加编号}
func (h PairHeap) Len() int           { return len(h) }
func (h PairHeap) Less(i, j int) bool { return h[i][0] < h[j][0] }
func (h PairHeap) Swap(i, j int)      { h[i], h[j] = h[j], h[i] }
func (h *PairHeap) Push(x any)        { *h = append(*h, x.([2]int64)) }
func (h *PairHeap) Pop() any          { a := *h; x := a[len(a)-1]; *h = a[:len(a)-1]; return x }
func CollectionsExample() {
	stack := []int{1}
	stack = append(stack, 2)
	stack = stack[:len(stack)-1]
	q := []int{1, 2}
	head := 0
	_ = q[head]
	head++ // 长期运行可定期整理已消费前缀
	h := &PairHeap{}
	heap.Push(h, [2]int64{3, 0})
	_ = heap.Pop(h).([2]int64)
	freq := map[int]int{}
	freq[3]++
	set := map[int]struct{}{3: {}}
	_, ok := set[3]
	_ = ok
}
```

## 单调栈与单调队列

左侧最近严格较小下标（不存在 -1）与窗口最小值，O(n)。窗口须满足 1≤k≤n。

```go
func PreviousLess(a []int64) []int {
	res := make([]int, len(a))
	st := []int{}
	for i, x := range a {
		for len(st) > 0 && a[st[len(st)-1]] >= x {
			st = st[:len(st)-1]
		}
		res[i] = -1
		if len(st) > 0 {
			res[i] = st[len(st)-1]
		}
		st = append(st, i)
	}
	return res
}
func WindowMin(a []int64, k int) []int64 {
	if k < 1 || k > len(a) {
		panic("invalid window")
	}
	q := make([]int, len(a))
	head, tail := 0, 0
	res := []int64{}
	for i, x := range a {
		for head < tail && q[head] <= i-k {
			head++
		}
		for head < tail && a[q[tail-1]] >= x {
			tail--
		}
		q[tail] = i
		tail++
		if i >= k-1 {
			res = append(res, a[q[head]])
		}
	}
	return res
}
```

## KMP 字符串匹配

返回全部匹配起点，支持重叠，空模式匹配每个边界。O(n+m)。Go/Rust 按 UTF-8 字节，TypeScript 按 UTF-16 码元编号。

```go
func KMP(text, pattern string) []int {
	n, m := len(text), len(pattern)
	res := []int{}
	if m == 0 {
		for i := 0; i <= n; i++ {
			res = append(res, i)
		}
		return res
	}
	pi := make([]int, m)
	for i, j := 1, 0; i < m; i++ {
		for j > 0 && pattern[i] != pattern[j] {
			j = pi[j-1]
		}
		if pattern[i] == pattern[j] {
			j++
		}
		pi[i] = j
	}
	for i, j := 0, 0; i < n; i++ {
		for j > 0 && text[i] != pattern[j] {
			j = pi[j-1]
		}
		if text[i] == pattern[j] {
			j++
		}
		if j == m {
			res = append(res, i-m+1)
			j = pi[j-1]
		}
	}
	return res
}
```

## Trie 字典树

插入单词并统计完整单词出现次数，O(单词长度)。这里限定小写英文字母。

```go
type TrieNode struct {
	Next  [26]*TrieNode
	Count int
}
type Trie struct{ Root TrieNode }

func (t *Trie) Insert(s string) {
	p := &t.Root
	for _, c := range []byte(s) {
		i := c - 'a'
		if p.Next[i] == nil {
			p.Next[i] = &TrieNode{}
		}
		p = p.Next[i]
	}
	p.Count++
}
func (t *Trie) Count(s string) int {
	p := &t.Root
	for _, c := range []byte(s) {
		p = p.Next[c-'a']
		if p == nil {
			return 0
		}
	}
	return p.Count
}
```

## 并查集

路径压缩与按集合大小合并，均摊 O(α(n))。重复合并不改变大小。

```go
type DSU struct{ Parent, Size []int }

func NewDSU(n int) *DSU {
	d := &DSU{make([]int, n), make([]int, n)}
	for i := 0; i < n; i++ {
		d.Parent[i] = i
		d.Size[i] = 1
	}
	return d
}
func (d *DSU) Find(x int) int {
	for d.Parent[x] != x {
		d.Parent[x] = d.Parent[d.Parent[x]]
		x = d.Parent[x]
	}
	return x
}
func (d *DSU) Union(a, b int) bool {
	a, b = d.Find(a), d.Find(b)
	if a == b {
		return false
	}
	if d.Size[a] < d.Size[b] {
		a, b = b, a
	}
	d.Parent[b] = a
	d.Size[a] += d.Size[b]
	return true
}
```

## 可修改、可删除的编号堆

编号唯一的最小堆，支持插入、按编号修改和删除，O(log n)。peek 返回 {值,编号}；操作须针对存在的编号，peek 前须非空。

```go
type IndexedHeap struct {
	Data [][2]int64
	Pos  map[int]int
}

func NewIndexedHeap() *IndexedHeap { return &IndexedHeap{Pos: map[int]int{}} }
func (h *IndexedHeap) swap(i, j int) {
	h.Data[i], h.Data[j] = h.Data[j], h.Data[i]
	h.Pos[int(h.Data[i][1])] = i
	h.Pos[int(h.Data[j][1])] = j
}
func (h *IndexedHeap) up(i int) int {
	for i > 0 {
		p := (i - 1) / 2
		if h.Data[p][0] <= h.Data[i][0] {
			break
		}
		h.swap(i, p)
		i = p
	}
	return i
}
func (h *IndexedHeap) down(i int) {
	for 2*i+1 < len(h.Data) {
		j := 2*i + 1
		if j+1 < len(h.Data) && h.Data[j+1][0] < h.Data[j][0] {
			j++
		}
		if h.Data[i][0] <= h.Data[j][0] {
			break
		}
		h.swap(i, j)
		i = j
	}
}
func (h *IndexedHeap) Insert(id int, v int64) {
	if _, ok := h.Pos[id]; ok {
		panic("duplicate id")
	}
	h.Pos[id] = len(h.Data)
	h.Data = append(h.Data, [2]int64{v, int64(id)})
	h.up(len(h.Data) - 1)
}
func (h *IndexedHeap) Change(id int, v int64) { i := h.Pos[id]; h.Data[i][0] = v; h.down(h.up(i)) }
func (h *IndexedHeap) Remove(id int) {
	i := h.Pos[id]
	last := len(h.Data) - 1
	h.swap(i, last)
	h.Data = h.Data[:last]
	delete(h.Pos, id)
	if i < len(h.Data) {
		h.down(h.up(i))
	}
}
func (h *IndexedHeap) Peek() [2]int64 { return h.Data[0] }
```

## 哈希：开放寻址与链表法

开放寻址与链表法整数集合，仅插入和查询。容量为正，开放寻址容量须大于不同元素数。独立占用标记支持任意键值。

```go
type OpenHash struct {
	Keys []int64
	Used []bool
}

func NewOpenHash(n int) *OpenHash { return &OpenHash{make([]int64, n), make([]bool, n)} }
func (h *OpenHash) locate(x int64) int {
	n := int64(len(h.Keys))
	k := int((x%n + n) % n)
	start := k
	for h.Used[k] && h.Keys[k] != x {
		k = (k + 1) % len(h.Keys)
		if k == start {
			panic("full table")
		}
	}
	return k
}
func (h *OpenHash) Add(x int64)           { k := h.locate(x); h.Keys[k] = x; h.Used[k] = true }
func (h *OpenHash) Contains(x int64) bool { return h.Used[h.locate(x)] }

type ChainHash struct{ Buckets [][]int64 }

func NewChainHash(n int) *ChainHash     { return &ChainHash{make([][]int64, n)} }
func (h *ChainHash) bucket(x int64) int { n := int64(len(h.Buckets)); return int((x%n + n) % n) }
func (h *ChainHash) Contains(x int64) bool {
	for _, v := range h.Buckets[h.bucket(x)] {
		if v == x {
			return true
		}
	}
	return false
}
func (h *ChainHash) Add(x int64) {
	if !h.Contains(x) {
		k := h.bucket(x)
		h.Buckets[k] = append(h.Buckets[k], x)
	}
}
```

## 字符串前缀哈希

前缀哈希查询 [l,r)，O(n) 构建、O(1) 查询。base=131，mod=1000000007，有碰撞可能；精确比较需再核对原字符串。Go/Rust 按字节，TypeScript 按 UTF-16 码元。

```go
type StringHash struct{ H, P []int64 }

func NewStringHash(s string) *StringHash {
	h := &StringHash{make([]int64, len(s)+1), make([]int64, len(s)+1)}
	h.P[0] = 1
	for i := 0; i < len(s); i++ {
		h.H[i+1] = (h.H[i]*131 + int64(s[i]) + 1) % 1000000007
		h.P[i+1] = h.P[i] * 131 % 1000000007
	}
	return h
}
func (h *StringHash) Get(l, r int) int64 {
	return (h.H[r] - h.H[l]*h.P[r-l]%1000000007 + 1000000007) % 1000000007
}
```

## 二叉树遍历

一般二叉树的前、中、后序遍历，O(n)。Go/TypeScript 使用节点引用；Rust 使用节点数组与 Option<usize> 子节点索引。树须无环，迭代遍历。

```go
type TreeNode struct {
	Value       int64
	Left, Right *TreeNode
}

func Traversals(root *TreeNode) ([]int64, []int64, []int64) {
	pre, in, post := []int64{}, []int64{}, []int64{}
	st := []*TreeNode{}
	if root != nil {
		st = append(st, root)
	}
	for len(st) > 0 {
		u := st[len(st)-1]
		st = st[:len(st)-1]
		pre = append(pre, u.Value)
		if u.Right != nil {
			st = append(st, u.Right)
		}
		if u.Left != nil {
			st = append(st, u.Left)
		}
	}
	cur := root
	for cur != nil || len(st) > 0 {
		for cur != nil {
			st = append(st, cur)
			cur = cur.Left
		}
		cur = st[len(st)-1]
		st = st[:len(st)-1]
		in = append(in, cur.Value)
		cur = cur.Right
	}
	if root != nil {
		st = append(st, root)
	}
	for len(st) > 0 {
		u := st[len(st)-1]
		st = st[:len(st)-1]
		post = append(post, u.Value)
		if u.Left != nil {
			st = append(st, u.Left)
		}
		if u.Right != nil {
			st = append(st, u.Right)
		}
	}
	for l, r := 0, len(post)-1; l < r; l, r = l+1, r-1 {
		post[l], post[r] = post[r], post[l]
	}
	return pre, in, post
}
```

## 哈夫曼树与合并代价

非负权值的最小合并代价，即哈夫曼带权路径长度。空集合或单节点为 0，O(n log n)。Go/TypeScript 依赖前文 PairHeap / Heap。

```go
func HuffmanCost(a []int64) int64 {
	h := &PairHeap{}
	for _, x := range a {
		heap.Push(h, [2]int64{x, 0})
	}
	var ans int64
	for h.Len() > 1 {
		s := heap.Pop(h).([2]int64)[0] + heap.Pop(h).([2]int64)[0]
		ans += s
		heap.Push(h, [2]int64{s, 0})
	}
	return ans
}
```

## 树状数组：点加、区间加与区间和

本节一基下标。点加、前缀和与区间和；双树状数组支持闭区间 [l,r] 加与区间和。初始为零，O(log n)。

```go
type Fenwick struct{ Tree []int64 }

func NewFenwick(n int) *Fenwick { return &Fenwick{make([]int64, n+1)} }
func (f *Fenwick) Add(i int, v int64) {
	if i <= 0 {
		panic("index starts at 1")
	}
	for i < len(f.Tree) {
		f.Tree[i] += v
		i += i & -i
	}
}
func (f *Fenwick) Prefix(i int) int64 {
	var s int64
	for i > 0 {
		s += f.Tree[i]
		i -= i & -i
	}
	return s
}
func (f *Fenwick) RangeSum(l, r int) int64 { return f.Prefix(r) - f.Prefix(l-1) }

type RangeFenwick struct{ B1, B2 *Fenwick }

func NewRangeFenwick(n int) *RangeFenwick { return &RangeFenwick{NewFenwick(n), NewFenwick(n)} }
func (f *RangeFenwick) RangeAdd(l, r int, v int64) {
	f.B1.Add(l, v)
	f.B1.Add(r+1, -v)
	f.B2.Add(l, v*int64(l-1))
	f.B2.Add(r+1, -v*int64(r))
}
func (f *RangeFenwick) Prefix(i int) int64      { return f.B1.Prefix(i)*int64(i) - f.B2.Prefix(i) }
func (f *RangeFenwick) RangeSum(l, r int) int64 { return f.Prefix(r) - f.Prefix(l-1) }
```

## 线段树：懒标记区间加、区间和

非空数组，零基闭区间 [l,r]。懒标记支持区间加与区间和，O(log n)。

```go
type SegmentTree struct {
	N         int
	Sum, Lazy []int64
}

func NewSegmentTree(a []int64) *SegmentTree {
	s := &SegmentTree{len(a), make([]int64, 4*len(a)), make([]int64, 4*len(a))}
	if len(a) == 0 {
		panic("empty array")
	}
	s.build(1, 0, len(a)-1, a)
	return s
}
func (s *SegmentTree) build(u, l, r int, a []int64) {
	if l == r {
		s.Sum[u] = a[l]
		return
	}
	m := (l + r) / 2
	s.build(u*2, l, m, a)
	s.build(u*2+1, m+1, r, a)
	s.Sum[u] = s.Sum[u*2] + s.Sum[u*2+1]
}
func (s *SegmentTree) apply(u, l, r int, v int64) { s.Sum[u] += int64(r-l+1) * v; s.Lazy[u] += v }
func (s *SegmentTree) push(u, l, r int) {
	m := (l + r) / 2
	s.apply(u*2, l, m, s.Lazy[u])
	s.apply(u*2+1, m+1, r, s.Lazy[u])
	s.Lazy[u] = 0
}
func (s *SegmentTree) Add(ql, qr int, v int64) { s.add(1, 0, s.N-1, ql, qr, v) }
func (s *SegmentTree) add(u, l, r, ql, qr int, v int64) {
	if ql <= l && r <= qr {
		s.apply(u, l, r, v)
		return
	}
	s.push(u, l, r)
	m := (l + r) / 2
	if ql <= m {
		s.add(u*2, l, m, ql, qr, v)
	}
	if qr > m {
		s.add(u*2+1, m+1, r, ql, qr, v)
	}
	s.Sum[u] = s.Sum[u*2] + s.Sum[u*2+1]
}
func (s *SegmentTree) Query(ql, qr int) int64 { return s.query(1, 0, s.N-1, ql, qr) }
func (s *SegmentTree) query(u, l, r, ql, qr int) int64 {
	if ql <= l && r <= qr {
		return s.Sum[u]
	}
	s.push(u, l, r)
	m := (l + r) / 2
	var ans int64
	if ql <= m {
		ans += s.query(u*2, l, m, ql, qr)
	}
	if qr > m {
		ans += s.query(u*2+1, m+1, r, ql, qr)
	}
	return ans
}
```

## 树形 DP：最大权独立集

非空无向树，最大权独立集：相邻节点不能同时选，可全不选。迭代建树后逆序转移，O(n)。

```go
func TreeIndependentSet(g [][]int, w []int64) int64 {
	n := len(g)
	p := make([]int, n)
	for i := range p {
		p[i] = -2
	}
	p[0] = -1
	order := []int{0}
	for i := 0; i < len(order); i++ {
		u := order[i]
		for _, v := range g[u] {
			if p[v] == -2 {
				p[v] = u
				order = append(order, v)
			}
		}
	}
	f := make([][2]int64, n)
	for i := n - 1; i >= 0; i-- {
		u := order[i]
		f[u][1] += w[u]
		if p[u] >= 0 {
			f[p[u]][0] += max(f[u][0], f[u][1])
			f[p[u]][1] += f[u][0]
		}
	}
	return max(f[0][0], f[0][1])
}
```

# 搜索与图论

## DFS：全排列与 N 皇后

返回全排列与 N 皇后的全部解；内存随解数增长，仅用于搜索深度较小的题。皇后解记录各行列号。

```go
func Permutations(n int) [][]int {
	ans := [][]int{}
	path := make([]int, n)
	used := make([]bool, n)
	var dfs func(int)
	dfs = func(d int) {
		if d == n {
			ans = append(ans, append([]int{}, path...))
			return
		}
		for x := 0; x < n; x++ {
			if !used[x] {
				used[x] = true
				path[d] = x
				dfs(d + 1)
				used[x] = false
			}
		}
	}
	dfs(0)
	return ans
}
func NQueens(n int) [][]int {
	ans := [][]int{}
	path := make([]int, n)
	col, d1, d2 := make([]bool, n), make([]bool, 2*n+1), make([]bool, 2*n+1)
	var dfs func(int)
	dfs = func(r int) {
		if r == n {
			ans = append(ans, append([]int{}, path...))
			return
		}
		for c := 0; c < n; c++ {
			if !col[c] && !d1[r+c] && !d2[r-c+n] {
				col[c], d1[r+c], d2[r-c+n] = true, true, true
				path[r] = c
				dfs(r + 1)
				col[c], d1[r+c], d2[r-c+n] = false, false, false
			}
		}
	}
	dfs(0)
	return ans
}
```

## 树的重心

树的重心指标：删除单个节点后最大连通块大小的最小值；非空无向树，O(n)。

```go
func CentroidBalance(g [][]int) int {
	n := len(g)
	p, size := make([]int, n), make([]int, n)
	for i := range p {
		p[i] = -2
	}
	p[0] = -1
	order := []int{0}
	for i := 0; i < len(order); i++ {
		u := order[i]
		for _, v := range g[u] {
			if p[v] == -2 {
				p[v] = u
				order = append(order, v)
			}
		}
	}
	best := n
	for i := n - 1; i >= 0; i-- {
		u := order[i]
		size[u] = 1
		largest := 0
		for _, v := range g[u] {
			if p[v] == u {
				size[u] += size[v]
				largest = max(largest, size[v])
			}
		}
		best = min(best, max(largest, n-size[u]))
	}
	return best
}
```

## BFS：图的层次与迷宫

无权图距离，不可达 -1；迷宫 0 可走、1 障碍，起终点为左上与右下。O(V+E) / O(nm)。

```go
func BFS(g [][]int, source int) []int {
	d := make([]int, len(g))
	for i := range d {
		d[i] = -1
	}
	d[source] = 0
	q := []int{source}
	for h := 0; h < len(q); h++ {
		u := q[h]
		for _, v := range g[u] {
			if d[v] == -1 {
				d[v] = d[u] + 1
				q = append(q, v)
			}
		}
	}
	return d
}
func MazeDistance(a [][]int) int {
	n := len(a)
	if n == 0 || len(a[0]) == 0 {
		return -1
	}
	m := len(a[0])
	if a[0][0] != 0 || a[n-1][m-1] != 0 {
		return -1
	}
	d := make([][]int, n)
	for i := range d {
		d[i] = make([]int, m)
		for j := range d[i] {
			d[i][j] = -1
		}
	}
	d[0][0] = 0
	q := [][2]int{{0, 0}}
	dirs := [][2]int{{-1, 0}, {0, 1}, {1, 0}, {0, -1}}
	for h := 0; h < len(q); h++ {
		u := q[h]
		for _, dir := range dirs {
			x, y := u[0]+dir[0], u[1]+dir[1]
			if x >= 0 && x < n && y >= 0 && y < m && a[x][y] == 0 && d[x][y] == -1 {
				d[x][y] = d[u[0]][u[1]] + 1
				q = append(q, [2]int{x, y})
			}
		}
	}
	return d[n-1][m-1]
}
```

## Dijkstra：朴素与堆优化

边权非负。矩阵无边用 INF（Go/Rust 的最大 i64 /4）或 Infinity；邻接项为 {终点,权重}。朴素 O(n²)，堆优化 O((V+E) log V)。Go/TypeScript 依赖 PairHeap / Heap。

```go
const INF int64 = 1<<61 - 1

type Edge struct {
	To int
	W  int64
}

func DijkstraDense(g [][]int64, s int) []int64 {
	n := len(g)
	d := make([]int64, n)
	used := make([]bool, n)
	for i := range d {
		d[i] = INF
	}
	d[s] = 0
	for i := 0; i < n; i++ {
		u := -1
		for j := 0; j < n; j++ {
			if !used[j] && (u == -1 || d[j] < d[u]) {
				u = j
			}
		}
		if u == -1 || d[u] == INF {
			break
		}
		used[u] = true
		for v, w := range g[u] {
			if w != INF {
				d[v] = min(d[v], d[u]+w)
			}
		}
	}
	return d
}
func Dijkstra(g [][]Edge, s int) []int64 {
	d := make([]int64, len(g))
	for i := range d {
		d[i] = INF
	}
	d[s] = 0
	h := &PairHeap{{0, int64(s)}}
	for h.Len() > 0 {
		item := heap.Pop(h).([2]int64)
		u := int(item[1])
		if item[0] != d[u] {
			continue
		}
		for _, e := range g[u] {
			if d[e.To] > d[u]+e.W {
				d[e.To] = d[u] + e.W
				heap.Push(h, [2]int64{d[e.To], int64(e.To)})
			}
		}
	}
	return d
}
```

## Bellman-Ford：最多 k 条边

边表 {u,v,w}，允许负权；最多 k 条边，每轮使用上一轮副本。不可达 INF / Infinity，O(kE)。Go/Rust 依赖前文 INF。

```go
type Arc struct {
	From, To int
	W        int64
}

func BellmanFord(n int, edges []Arc, s, k int) []int64 {
	d := make([]int64, n)
	for i := range d {
		d[i] = INF
	}
	d[s] = 0
	for i := 0; i < k; i++ {
		old := append([]int64(nil), d...)
		for _, e := range edges {
			if old[e.From] != INF {
				d[e.To] = min(d[e.To], old[e.From]+e.W)
			}
		}
	}
	return d
}
```

## SPFA

允许负权；源点可达负环时 Go 返回 false、TypeScript 抛异常、Rust 返回 None。不可达 INF / Infinity，最坏 O(VE)。依赖 Edge 与 INF（TypeScript 不需要 INF）。

```go
func SPFA(g [][]Edge, s int) ([]int64, bool) {
	n := len(g)
	d := make([]int64, n)
	cnt := make([]int, n)
	in := make([]bool, n)
	for i := range d {
		d[i] = INF
	}
	d[s] = 0
	q := []int{s}
	in[s] = true
	for head := 0; head < len(q); head++ {
		u := q[head]
		in[u] = false
		for _, e := range g[u] {
			v := e.To
			if d[v] > d[u]+e.W {
				d[v] = d[u] + e.W
				cnt[v] = cnt[u] + 1
				if cnt[v] >= n {
					return nil, false
				}
				if !in[v] {
					in[v] = true
					q = append(q, v)
				}
			}
		}
	}
	return d, true
}
```

## Floyd 全源最短路

无负环；对角线 0，无边 INF / Infinity，重边建图取最小值。返回副本，O(n³)。Go/Rust 依赖 INF。

```go
func Floyd(g [][]int64) [][]int64 {
	n := len(g)
	d := make([][]int64, n)
	for i := range d {
		d[i] = append([]int64(nil), g[i]...)
	}
	for k := 0; k < n; k++ {
		for i := 0; i < n; i++ {
			if d[i][k] != INF {
				for j := 0; j < n; j++ {
					if d[k][j] != INF {
						d[i][j] = min(d[i][j], d[i][k]+d[k][j])
					}
				}
			}
		}
	}
	return d
}
```

## 最小生成树：Prim 与 Kruskal

无向图，允许负权。Prim 使用矩阵（无边 INF / Infinity），O(n²)；Kruskal 边表使用 Arc，O(E log E)，依赖 DSU。非连通为 Go false、TS null、Rust None，空图代价 0。

```go
func Prim(g [][]int64) (int64, bool) {
	n := len(g)
	if n == 0 {
		return 0, true
	}
	d := make([]int64, n)
	used := make([]bool, n)
	for i := range d {
		d[i] = INF
	}
	d[0] = 0
	var ans int64
	for i := 0; i < n; i++ {
		u := -1
		for j := 0; j < n; j++ {
			if !used[j] && (u == -1 || d[j] < d[u]) {
				u = j
			}
		}
		if d[u] == INF {
			return 0, false
		}
		ans += d[u]
		used[u] = true
		for v, w := range g[u] {
			if !used[v] {
				d[v] = min(d[v], w)
			}
		}
	}
	return ans, true
}
func Kruskal(n int, edges []Arc) (int64, bool) {
	a := append([]Arc(nil), edges...)
	sort.Slice(a, func(i, j int) bool { return a[i].W < a[j].W })
	d := NewDSU(n)
	var ans int64
	count := 0
	for _, e := range a {
		if d.Union(e.From, e.To) {
			ans += e.W
			count++
		}
	}
	return ans, n == 0 || count == n-1
}
```

## 二分图：染色与匈牙利算法

无向图染色 O(V+E)。最大匹配输入仅左点邻接表，邻接值为右点编号；迭代增广 O(VE)，不受递归栈深度限制。

```go
func IsBipartite(g [][]int) bool {
	color := make([]int, len(g))
	for s := range g {
		if color[s] != 0 {
			continue
		}
		color[s] = 1
		q := []int{s}
		for h := 0; h < len(q); h++ {
			u := q[h]
			for _, v := range g[u] {
				if color[v] == color[u] {
					return false
				}
				if color[v] == 0 {
					color[v] = 3 - color[u]
					q = append(q, v)
				}
			}
		}
	}
	return true
}
func MaxMatching(g [][]int, nr int) int {
	left, right := make([]int, len(g)), make([]int, nr)
	for i := range left {
		left[i] = -1
	}
	for i := range right {
		right[i] = -1
	}
	ans := 0
	for s := range g {
		prev := make([]int, nr)
		for i := range prev {
			prev[i] = -1
		}
		seen := make([]bool, len(g))
		seen[s] = true
		q := []int{s}
		end := -1
		for h := 0; h < len(q) && end == -1; h++ {
			u := q[h]
			for _, v := range g[u] {
				if prev[v] != -1 {
					continue
				}
				prev[v] = u
				if right[v] == -1 {
					end = v
					break
				}
				if !seen[right[v]] {
					seen[right[v]] = true
					q = append(q, right[v])
				}
			}
		}
		if end != -1 {
			ans++
			for end != -1 {
				u := prev[end]
				old := left[u]
				left[u] = end
				right[end] = u
				end = old
			}
		}
	}
	return ans
}
```

## 拓扑排序

有向图拓扑排序，O(V+E)。存在环时 Go 返回 false、TypeScript null、Rust None。

```go
func TopologicalSort(g [][]int) ([]int, bool) {
	deg := make([]int, len(g))
	for _, row := range g {
		for _, v := range row {
			deg[v]++
		}
	}
	q := []int{}
	for v, d := range deg {
		if d == 0 {
			q = append(q, v)
		}
	}
	for h := 0; h < len(q); h++ {
		for _, v := range g[q[h]] {
			deg[v]--
			if deg[v] == 0 {
				q = append(q, v)
			}
		}
	}
	return q, len(q) == len(g)
}
```

# 数学知识

## 质数：试除、分解与筛法

试除 O(√n)，分解返回质因子指数表（n≥1）。两种筛均包含上界 n：埃氏 O(n log log n)、线性 O(n)。

```go
func IsPrime(n int64) bool {
	if n < 2 {
		return false
	}
	for p := int64(2); p <= n/p; p++ {
		if n%p == 0 {
			return false
		}
	}
	return true
}
func Factorize(n int64) map[int64]int {
	if n < 1 {
		panic("positive required")
	}
	f := map[int64]int{}
	for p := int64(2); p <= n/p; p++ {
		for n%p == 0 {
			f[p]++
			n /= p
		}
	}
	if n > 1 {
		f[n]++
	}
	return f
}
func Sieve(n int) []int {
	comp := make([]bool, n+1)
	primes := []int{}
	for i := 2; i <= n; i++ {
		if !comp[i] {
			primes = append(primes, i)
			if i <= n/i {
				for j := i * i; j <= n; j += i {
					comp[j] = true
				}
			}
		}
	}
	return primes
}
func LinearSieve(n int) []int {
	comp := make([]bool, n+1)
	primes := []int{}
	for i := 2; i <= n; i++ {
		if !comp[i] {
			primes = append(primes, i)
		}
		for _, p := range primes {
			if p > n/i {
				break
			}
			comp[i*p] = true
			if i%p == 0 {
				break
			}
		}
	}
	return primes
}
```

## 约数、约数个数与约数之和

枚举正整数约数并排序；约数个数与和从质因子指数表计算。模数须正；Go 限定 mod≤2147483647，Rust 模数用 u32 并用 u128 中间乘法，TypeScript 用 bigint。

```go
func Divisors(n int64) []int64 {
	if n < 1 {
		panic("positive required")
	}
	a := []int64{}
	for i := int64(1); i <= n/i; i++ {
		if n%i == 0 {
			a = append(a, i)
			if i != n/i {
				a = append(a, n/i)
			}
		}
	}
	sort.Slice(a, func(i, j int) bool { return a[i] < a[j] })
	return a
}
func DivisorCount(f map[int64]int, mod int64) int64 {
	ans := int64(1) % mod
	for _, e := range f {
		ans = ans * int64(e+1) % mod
	}
	return ans
}
func DivisorSum(f map[int64]int, mod int64) int64 {
	ans := int64(1) % mod
	for p, e := range f {
		term := int64(1) % mod
		for i := 0; i < e; i++ {
			term = (term*(p%mod) + 1) % mod
		}
		ans = ans * term % mod
	}
	return ans
}
```

## 欧几里得、扩展欧几里得与线性同余方程

a、b 非负且不同时为 0；扩展欧几里得返回 (g,x,y)，ax+by=g。线性同余方程 ax≡b(mod m)，m>0；无解为 false / null / None。Go/Rust 系数中间值须在 i64 范围内，TS 使用 bigint。

```go
func GCD(a, b int64) int64 {
	for b != 0 {
		a, b = b, a%b
	}
	return a
}
func ExGCD(a, b int64) (int64, int64, int64) {
	if b == 0 {
		return a, 1, 0
	}
	g, x, y := ExGCD(b, a%b)
	return g, y, x - a/b*y
}
func SolveCongruence(a, b, m int64) (int64, bool) {
	g, x, _ := ExGCD(a, m)
	if b%g != 0 {
		return 0, false
	}
	period := m / g
	r := x * (b / g) % period
	if r < 0 {
		r += period
	}
	return r, true
}
```

## 快速幂

指数非负，模数正，O(log b)。Go 模数≤2147483647；Rust 模数 u32，乘法用 u64；TypeScript 使用 bigint。

```go
func ModPow(a, b, mod int64) int64 {
	a = (a%mod + mod) % mod
	res := int64(1) % mod
	for b > 0 {
		if b&1 != 0 {
			res = res * a % mod
		}
		a = a * a % mod
		b >>= 1
	}
	return res
}
```

## 高斯消元

n×(n+1) 增广矩阵，直接修改；0=唯一解、1=无穷多解、2=无解。唯一解存最后一列，O(n³)，eps=1e-9。

```go
func Gauss(a [][]float64) int {
	n, r := len(a), 0
	const eps = 1e-9
	for c := 0; c < n; c++ {
		t := r
		for i := r; i < n; i++ {
			if math.Abs(a[i][c]) > math.Abs(a[t][c]) {
				t = i
			}
		}
		if math.Abs(a[t][c]) < eps {
			continue
		}
		a[t], a[r] = a[r], a[t]
		for j := n; j >= c; j-- {
			a[r][j] /= a[r][c]
		}
		for i := r + 1; i < n; i++ {
			if math.Abs(a[i][c]) > eps {
				for j := n; j >= c; j-- {
					a[i][j] -= a[r][j] * a[i][c]
				}
			}
		}
		r++
	}
	if r < n {
		for i := r; i < n; i++ {
			if math.Abs(a[i][n]) > eps {
				return 2
			}
		}
		return 1
	}
	for i := n - 1; i >= 0; i-- {
		for j := i + 1; j < n; j++ {
			a[i][n] -= a[i][j] * a[j][n]
		}
	}
	return 0
}
```

## 组合数：递推、阶乘逆元、Lucas 与精确值

递推适用任意正模数。阶乘逆元要求 p 为质数、n<p；Lucas 要求 p 为质数且适合较小 p，小组合 O(p)。依赖 ModPow / modPow / mod_pow。Go 模数≤2147483647；Rust 用 u32；精确组合数 Go 用 math/big、TS 用 bigint，Rust 返回十进制字符串并依赖后文数位运算。

```go
func CombinationTable(n int, mod int64) [][]int64 {
	c := make([][]int64, n+1)
	for i := range c {
		c[i] = make([]int64, n+1)
		c[i][0] = 1 % mod
		for j := 1; j <= i; j++ {
			c[i][j] = (c[i-1][j-1] + c[i-1][j]) % mod
		}
	}
	return c
}

type CombPrime struct {
	Fact, Inv []int64
	P         int64
}

func NewCombPrime(n int, p int64) *CombPrime {
	if p < 2 || int64(n) >= p {
		panic("require n < prime p")
	}
	c := &CombPrime{make([]int64, n+1), make([]int64, n+1), p}
	c.Fact[0] = 1
	for i := 1; i <= n; i++ {
		c.Fact[i] = c.Fact[i-1] * int64(i) % p
	}
	c.Inv[n] = ModPow(c.Fact[n], p-2, p)
	for i := n; i > 0; i-- {
		c.Inv[i-1] = c.Inv[i] * int64(i) % p
	}
	return c
}
func (c *CombPrime) Choose(n, k int) int64 {
	if k < 0 || k > n {
		return 0
	}
	return c.Fact[n] * c.Inv[k] % c.P * c.Inv[n-k] % c.P
}
func SmallComb(n, k, p int64) int64 {
	if k > n {
		return 0
	}
	k = min(k, n-k)
	a, b := int64(1), int64(1)
	for i := int64(1); i <= k; i++ {
		a = a * (n - i + 1) % p
		b = b * i % p
	}
	return a * ModPow(b, p-2, p) % p
}
func Lucas(n, k, p int64) int64 {
	if p < 2 {
		panic("prime p required")
	}
	ans := int64(1)
	for n > 0 || k > 0 {
		ans = ans * SmallComb(n%p, k%p, p) % p
		n /= p
		k /= p
	}
	return ans
}
func ExactComb(n, k int64) *big.Int {
	if k < 0 || k > n {
		return new(big.Int)
	}
	return new(big.Int).Binomial(n, k)
}
```

## 卡特兰数

返回精确卡特兰数 C(2n,n)/(n+1)，n≥0。依赖精确组合数；Rust 还依赖后文数位运算，n 须使 2n 可用 u32 表示。

```go
func Catalan(n int64) *big.Int { return new(big.Int).Quo(ExactComb(2*n, n), big.NewInt(n+1)) }
```

# 贪心

## 合并果子、区间覆盖、分组与选点

合并果子调用前文哈夫曼代价。闭区间覆盖要求 start<end，失败返回 -1；区间分组端点相接冲突；选点可用公共端点覆盖多段。O(n log n)，Go/TS 分组依赖最小堆。

```go
func CoverIntervals(start, end int64, ranges [][2]int64) int {
	a := append([][2]int64(nil), ranges...)
	sort.Slice(a, func(i, j int) bool { return a[i][0] < a[j][0] })
	i, count := 0, 0
	for start < end {
		next := start
		for i < len(a) && a[i][0] <= start {
			next = max(next, a[i][1])
			i++
		}
		if next == start {
			return -1
		}
		start = next
		count++
	}
	return count
}
func GroupIntervals(ranges [][2]int64) int {
	a := append([][2]int64(nil), ranges...)
	sort.Slice(a, func(i, j int) bool { return a[i][0] < a[j][0] })
	h := &PairHeap{}
	for _, r := range a {
		if h.Len() > 0 && (*h)[0][0] < r[0] {
			heap.Pop(h)
		}
		heap.Push(h, [2]int64{r[1], 0})
	}
	return h.Len()
}
func SelectPoints(ranges [][2]int64) int {
	a := append([][2]int64(nil), ranges...)
	sort.Slice(a, func(i, j int) bool { return a[i][1] < a[j][1] })
	var point int64
	count := 0
	for _, r := range a {
		if count == 0 || point < r[0] {
			point = r[1]
			count++
		}
	}
	return count
}
```

# 动态规划

## 背包：01、完全、多重与分组

容量至多 m，可不选；体积为正、数量非负，价值可负。01/完全 O(nm)，多重二进制拆分 O(mΣlog(s+1))，分组 O(mΣ组大小)。

```go
type Item struct {
	Volume int
	Value  int64
}
type BoundedItem struct {
	Volume int
	Value  int64
	Count  int
}

func Knapsack01(m int, items []Item) int64 {
	f := make([]int64, m+1)
	for _, x := range items {
		for j := m; j >= x.Volume; j-- {
			f[j] = max(f[j], f[j-x.Volume]+x.Value)
		}
	}
	return f[m]
}
func KnapsackComplete(m int, items []Item) int64 {
	f := make([]int64, m+1)
	for _, x := range items {
		for j := x.Volume; j <= m; j++ {
			f[j] = max(f[j], f[j-x.Volume]+x.Value)
		}
	}
	return f[m]
}
func KnapsackBounded(m int, items []BoundedItem) int64 {
	f := make([]int64, m+1)
	for _, x := range items {
		remaining := min(x.Count, m/x.Volume)
		for k := 1; remaining > 0; {
			take := min(k, remaining)
			remaining -= take
			v, w := x.Volume*take, x.Value*int64(take)
			for j := m; j >= v; j-- {
				f[j] = max(f[j], f[j-v]+w)
			}
			if remaining == 0 {
				break
			}
			k = min(k*2, remaining)
		}
	}
	return f[m]
}
func KnapsackGrouped(m int, groups [][]Item) int64 {
	f := make([]int64, m+1)
	for _, group := range groups {
		old := append([]int64(nil), f...)
		for j := 0; j <= m; j++ {
			for _, x := range group {
				if j >= x.Volume {
					f[j] = max(f[j], old[j-x.Volume]+x.Value)
				}
			}
		}
	}
	return f[m]
}
```

## 线性 DP：数字三角形

三角形第 i 行 i+1 项，最大路径和，可负数；自底向上 O(n²) 时间、O(n) 额外空间。空输入返回 0。

```go
func TriangleMax(a [][]int64) int64 {
	if len(a) == 0 {
		return 0
	}
	f := append([]int64(nil), a[len(a)-1]...)
	for i := len(a) - 2; i >= 0; i-- {
		for j := 0; j <= i; j++ {
			f[j] = a[i][j] + max(f[j], f[j+1])
		}
	}
	return f[0]
}
```

## 最长上升子序列

严格上升，O(n²) 返回一条正向序列；O(n log n) 版本只返回长度。空输入安全。

```go
func LIS(a []int64) []int64 {
	n := len(a)
	if n == 0 {
		return []int64{}
	}
	f, p := make([]int, n), make([]int, n)
	best := 0
	for i := range a {
		f[i] = 1
		p[i] = -1
		for j := 0; j < i; j++ {
			if a[j] < a[i] && f[j]+1 > f[i] {
				f[i] = f[j] + 1
				p[i] = j
			}
		}
		if f[i] > f[best] {
			best = i
		}
	}
	ans := make([]int64, f[best])
	for i := len(ans) - 1; i >= 0; i-- {
		ans[i] = a[best]
		best = p[best]
	}
	return ans
}
func LISLength(a []int64) int {
	tails := []int64{}
	for _, x := range a {
		i := sort.Search(len(tails), func(i int) bool { return tails[i] >= x })
		if i == len(tails) {
			tails = append(tails, x)
		} else {
			tails[i] = x
		}
	}
	return len(tails)
}
```

## 最长公共子序列

最长公共子序列长度，O(nm) 时间、O(m) 空间。Go/Rust 按字节，TypeScript 按 UTF-16 码元比较。

```go
func LCSLength(a, b string) int {
	f := make([]int, len(b)+1)
	for i := 0; i < len(a); i++ {
		diagonal := 0
		for j := 1; j <= len(b); j++ {
			old := f[j]
			if a[i] == b[j-1] {
				f[j] = diagonal + 1
			} else {
				f[j] = max(f[j], f[j-1])
			}
			diagonal = old
		}
	}
	return f[len(b)]
}
```

## 区间 DP：石子合并

线性排列的石子，每次合并相邻两堆，求最小总代价。O(n³) 时间、O(n²) 空间。

```go
func StoneMerge(a []int64) int64 {
	n := len(a)
	if n == 0 {
		return 0
	}
	s := Prefix(a)
	f := make([][]int64, n)
	for i := range f {
		f[i] = make([]int64, n)
	}
	for length := 2; length <= n; length++ {
		for l := 0; l+length <= n; l++ {
			r := l + length - 1
			f[l][r] = INF
			for k := l; k < r; k++ {
				f[l][r] = min(f[l][r], f[l][k]+f[k+1][r]+s[r+1]-s[l])
			}
		}
	}
	return f[0][n-1]
}
```

# 高精度

## 大整数：加、减、乘、除

十进制非负整数的加、减、乘、除；减法允许负结果，除数必须正。Go 使用 math/big；TS 使用 bigint；Rust 依赖下一节数位运算，不需外部 crate。除法返回商与余数，均为字符串。

```go
func ParseBig(s string) *big.Int {
	x, ok := new(big.Int).SetString(s, 10)
	if !ok || x.Sign() < 0 {
		panic("nonnegative decimal required")
	}
	return x
}
func BigAdd(a, b string) string      { return new(big.Int).Add(ParseBig(a), ParseBig(b)).String() }
func BigSubtract(a, b string) string { return new(big.Int).Sub(ParseBig(a), ParseBig(b)).String() }
func BigMultiply(a, b string) string { return new(big.Int).Mul(ParseBig(a), ParseBig(b)).String() }
func BigDivide(a, b string) (string, string) {
	d := ParseBig(b)
	if d.Sign() == 0 {
		panic("positive divisor required")
	}
	q, r := new(big.Int), new(big.Int)
	q.QuoRem(ParseBig(a), d, r)
	return q.String(), r.String()
}
```

## 手写高精度：十进制数位数组

数位低位在前，至少一位，无多余高位零。加法、非负减法（A≥B）、乘非负小整数、除正小整数。Go/TS 小整数≤10^9；Go 中间值 int64、TS 中间值仍在安全整数内。Rust 小整数 u64，中间值 u128。

```go
func Digits(s string) []int64 {
	a := make([]int64, len(s))
	for i := range a {
		a[i] = int64(s[len(s)-1-i] - '0')
	}
	return TrimDigits(a)
}
func TrimDigits(a []int64) []int64 {
	for len(a) > 1 && a[len(a)-1] == 0 {
		a = a[:len(a)-1]
	}
	return a
}
func AddDigits(a, b []int64) []int64 {
	c := []int64{}
	carry := int64(0)
	for i := 0; i < max(len(a), len(b)) || carry > 0; i++ {
		if i < len(a) {
			carry += a[i]
		}
		if i < len(b) {
			carry += b[i]
		}
		c = append(c, carry%10)
		carry /= 10
	}
	return c
}
func SubtractDigits(a, b []int64) []int64 {
	c := make([]int64, len(a))
	var borrow int64
	for i, x := range a {
		x -= borrow
		if i < len(b) {
			x -= b[i]
		}
		borrow = 0
		if x < 0 {
			x += 10
			borrow = 1
		}
		c[i] = x
	}
	return TrimDigits(c)
}
func MultiplyDigits(a []int64, b int64) []int64 {
	c := []int64{}
	var carry int64
	for i := 0; i < len(a) || carry > 0; i++ {
		if i < len(a) {
			carry += a[i] * b
		}
		c = append(c, carry%10)
		carry /= 10
	}
	return TrimDigits(c)
}
func DivideDigits(a []int64, b int64) ([]int64, int64) {
	c := make([]int64, len(a))
	var r int64
	for i := len(a) - 1; i >= 0; i-- {
		r = r*10 + a[i]
		c[i] = r / b
		r %= b
	}
	return TrimDigits(c), r
}
```

# 莫队

## 普通莫队：区间不同元素数

普通莫队，区间不同元素数；零基闭区间 [l,r]，原查询顺序返回。内部窗口使用 [L,R)；右端点奇偶块排序，典型 O((n+q)√n)。

```go
func MoAdjust(f map[int64]int, x int64, delta int) {
	f[x] += delta
	if f[x] == 0 {
		delete(f, x)
	}
}
func MoDistinct(a []int64, queries [][2]int) []int {
	b := max(1, int(math.Sqrt(float64(len(a)))))
	order := make([]int, len(queries))
	for i := range order {
		order[i] = i
	}
	sort.Slice(order, func(i, j int) bool {
		x, y := queries[order[i]], queries[order[j]]
		if x[0]/b != y[0]/b {
			return x[0]/b < y[0]/b
		}
		if x[0]/b%2 == 0 {
			return x[1] < y[1]
		}
		return x[1] > y[1]
	})
	f := map[int64]int{}
	l, r := 0, 0
	ans := make([]int, len(queries))
	for _, id := range order {
		ql, qr := queries[id][0], queries[id][1]+1
		for l > ql {
			l--
			MoAdjust(f, a[l], 1)
		}
		for r < qr {
			MoAdjust(f, a[r], 1)
			r++
		}
		for l < ql {
			MoAdjust(f, a[l], -1)
			l++
		}
		for r > qr {
			r--
			MoAdjust(f, a[r], -1)
		}
		ans[id] = len(f)
	}
	return ans
}
```

## 带修莫队：区间不同元素数

带单点赋值的区间不同元素数，零基闭区间。Go 操作 Kind=0 查询 [X,Y]、Kind=1 把 X 赋为 Y；TS 使用 [kind,x,y]；Rust 使用 Operation 枚举。按查询出现顺序返回，时间回滚交换旧值和新值。典型 O(n^(5/3))，依赖上一节频率更新。

```go
type Operation struct {
	Kind, X int
	Y       int64
}

func MoDistinctWithUpdates(initial []int64, ops []Operation) []int {
	a := append([]int64(nil), initial...)
	b := max(1, int(math.Pow(float64(max(1, len(a))), 2.0/3)))
	changes := [][2]int64{}
	queries := [][4]int{}
	for _, op := range ops {
		if op.Kind == 0 {
			queries = append(queries, [4]int{op.X, int(op.Y) + 1, len(changes), len(queries)})
		} else {
			changes = append(changes, [2]int64{int64(op.X), op.Y})
		}
	}
	sort.Slice(queries, func(i, j int) bool {
		x, y := queries[i], queries[j]
		if x[0]/b != y[0]/b {
			return x[0]/b < y[0]/b
		}
		if x[1]/b != y[1]/b {
			return x[1]/b < y[1]/b
		}
		return x[2] < y[2]
	})
	f := map[int64]int{}
	l, r, t := 0, 0, 0
	ans := make([]int, len(queries))
	change := func(i int) {
		p, v := int(changes[i][0]), changes[i][1]
		if l <= p && p < r {
			MoAdjust(f, a[p], -1)
			MoAdjust(f, v, 1)
		}
		a[p], changes[i][1] = v, a[p]
	}
	for _, q := range queries {
		for t < q[2] {
			change(t)
			t++
		}
		for t > q[2] {
			t--
			change(t)
		}
		for l > q[0] {
			l--
			MoAdjust(f, a[l], 1)
		}
		for r < q[1] {
			MoAdjust(f, a[r], 1)
			r++
		}
		for l < q[0] {
			MoAdjust(f, a[l], -1)
			l++
		}
		for r > q[1] {
			r--
			MoAdjust(f, a[r], -1)
		}
		ans[q[3]] = len(f)
	}
	return ans
}
```

