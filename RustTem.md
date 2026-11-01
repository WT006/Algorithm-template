# Rust 算法模板

与 [C++](./CppTem.md)、[Java](./JavaTem.md)、[Python](./PythonTem.md) 按知识点对应的 49 个算法小节。导航：[README](./README.md) · [C++](./CppTem.md) · [Java](./JavaTem.md) · [Python](./PythonTem.md) · [Go](./GoTem.md) · [TypeScript](./TypeScriptTem.md) · [Rust](./RustTem.md)。

## 使用约定

- 默认数组、图节点从 **0** 开始；树状数组从 **1** 开始。区间开闭、适用条件与依赖见各节。只复制所需实现与依赖，再补题目输入输出。
- 输入须满足有效下标与各节前提；浮点高斯消元使用容差。哈希比较可能碰撞。
- 使用 Rust 2021 edition 与标准库，无第三方 crate。索引用 `usize`，普通整数和、距离用 `i64`；中间结果也须在范围内。
- 各节定义放在模块顶层，按需加入以下导入，并补 `fn main()`。包含 Option 的返回值显式区分不可达与合法负值。

```rust
use std::collections::{
    HashMap, HashSet, VecDeque, BinaryHeap
} ;
use std::cmp::Reverse;
fn main() {
    // 调用选用算法。
}
```

## 内容导航

[基础算法](#基础算法) · [数据结构](#数据结构) · [搜索与图论](#搜索与图论) · [数学知识](#数学知识) · [贪心](#贪心) · [动态规划](#动态规划) · [高精度](#高精度) · [莫队](#莫队)

# 基础算法

## 快速输入

批量读取整数。Go 返回读取或解析错误；TypeScript 的解析函数接收输入字符串；Rust 读取标准输入。输入含 ASCII 空白，整数须在对应类型范围内。

```rust
fn read_ints() -> Result<Vec<i64>, Box<dyn std::error::Error>> {
    use std::io::Read;
    let mut s = String::new();
    std::io::stdin().read_to_string(&mut s)?;
    Ok(s.split_whitespace().map(str::parse).collect::<Result<Vec<i64>, _>>()?)
}
```

## 排序：快速排序与归并排序

快速排序原地修改，最坏 O(n²)；归并排序 O(n log n)。Rust 切片版空数组安全。

```rust
fn quick_sort(a: &mut [i64]) {
    if a.len() < 2 {
        return;
    }
    let (mut i, mut j, x) = (-1isize, a.len() as isize, a[(a.len()-1)/2]);
    while i < j {
        i += 1;
        while a[i as usize] < x {
            i += 1;
        }
        j -= 1;
        while a[j as usize] > x {
            j -= 1;
        }
        if i < j {
            a.swap(i as usize, j as usize);
        }
    }
    let (l, r) = a.split_at_mut(j as usize + 1);
    quick_sort(l);
    quick_sort(r);
}
fn merge_sort(a: &mut [i64]) {
    if a.len() < 2 {
        return;
    }
    let m = a.len()/2;
    merge_sort(&mut a[..m]);
    merge_sort(&mut a[m..]);
    let (mut i, mut j, mut tmp) = (0, m, Vec::with_capacity(a.len()));
    while i < m && j < a.len() {
        if a[i] <= a[j] {
            tmp.push(a[i]);
            i += 1;
        }
        else {
            tmp.push(a[j]);
            j += 1;
        }
    }
    tmp.extend_from_slice(&a[i..m]);
    tmp.extend_from_slice(&a[j..]);
    a.copy_from_slice(&tmp);
}
```

## 前缀和：一维与二维

一维查询 [l,r)，二维查询 [x1,x2)×[y1,y2)。O(n)/O(nm) 构建，O(1) 查询。

```rust
fn prefix(a: &[i64]) -> Vec<i64> {
    let mut s = vec![0;
    a.len()+1];
    for i in 0..a.len() {
        s[i+1]=s[i]+a[i];
    }
    s
}
fn prefix_2d(a: &[Vec<i64>]) -> Vec<Vec<i64>> {
    let (n,m)=(a.len(),a.first().map_or(0,Vec::len));
    let mut s=vec![vec![0;
    m+1];
    n+1];
    for i in 0..n {
        for j in 0..m {
            s[i+1][j+1]=a[i][j]+s[i][j+1]+s[i+1][j]-s[i][j];
        }
    }
    s
}
fn rect_sum(s:&[Vec<i64>],x1:usize,y1:usize,x2:usize,y2:usize)->i64 {
    s[x2][y2]-s[x1][y2]-s[x2][y1]+s[x1][y1]
}
```

## 差分：一维与二维

在零数组上区间加，更新左闭右开；差分数组比原数组多一位，二维多一行一列。

```rust
fn range_add(d:&mut [i64],l:usize,r:usize,v:i64) {
    d[l]+=v;
    d[r]-=v;
}
fn restore(d:&[i64])->Vec<i64> {
    let mut s=0;
    d[..d.len()-1].iter().map(|x| {
        s+=x; s
    }     ).collect()
}
fn rect_add(d:&mut [Vec<i64>],x1:usize,y1:usize,x2:usize,y2:usize,v:i64) {
    d[x1][y1]+=v;
    d[x2][y1]-=v;
    d[x1][y2]-=v;
    d[x2][y2]+=v;
}
fn restore_2d(d:&[Vec<i64>])->Vec<Vec<i64>> {
    let (n,m)=(d.len()-1,d[0].len()-1);
    let mut a=vec![vec![0;
    m];
    n];
    for i in 0..n {
        for j in 0..m {
            a[i][j]=d[i][j];
            if i>0 {
                a[i][j]+=a[i-1][j];
            }
            if j>0 {
                a[i][j]+=a[i][j-1];
            }
            if i>0&&j>0 {
                a[i][j]-=a[i-1][j-1];
            }
        }
    }
    a
}
```

## 二分：整数边界与浮点数

有序数组第一个 ≥x / >x 的位置，不存在返回 n；浮点二分求有限非负数平方根。

```rust
fn lower_bound(a:&[i64],x:i64)->usize {
    let(mut l,mut r)=(0,a.len());
    while l<r{
        let m=(l+r)/2;
        if a[m]>=x{
            r=m;
        }
        else{
            l=m+1;
        }
    }
    l
}
fn upper_bound(a:&[i64],x:i64)->usize {
    let(mut l,mut r)=(0,a.len());
    while l<r{
        let m=(l+r)/2;
        if a[m]>x{
            r=m;
        }
        else{
            l=m+1;
        }
    }
    l
}
fn sqrt_binary(x:f64)->f64 {
    assert!(x>=0.0&&x.is_finite());
    let(mut l,mut r)=(0.0,x.max(1.0));
    for _ in 0..1100{
        let m=l+(r-l)/2.0;
        if m==0.0||m<=x/m{
            l=m;
        }
        else{
            r=m;
        }
    }
    l
}
```

## 位运算

lowbit 输入正整数，popcount 统计非负整数二进制 1 的数量。TypeScript 使用 bigint，避免位运算隐式截断为 32 位。

```rust
fn lowbit(x:u64)->u64 {
    x & x.wrapping_neg()
}
fn popcount(x:u64)->u32 {
    x.count_ones()
}
```

## 区间合并

闭区间合并，端点相接视为重叠。O(n log n)，不修改原数组。

```rust
fn merge_intervals(ranges:&[(i64,i64)])->Vec<(i64,i64)>{
    let mut a=ranges.to_vec();
    a.sort_unstable();
    let mut res:Vec<(i64,i64)>=vec![];
    for (l,r) in a {
        if res.last().map_or(true,|x|x.1<l){
            res.push((l,r));
        }
        else{
            let x=res.last_mut().unwrap();
            x.1=x.1.max(r);
        }
    }
    res
}
```

## 离散化

返回排序去重后的值与原数组的零基排名，O(n log n)。

```rust
fn compress(a:&[i64])->(Vec<i64>,Vec<usize>){
    let mut values=a.to_vec();
    values.sort_unstable();
    values.dedup();
    let ranks=a.iter().map(|x|values.binary_search(x).unwrap()).collect();
    (values,ranks)
}
```

# 数据结构

## 数组模拟单链表与双链表

数组模拟链表。0 为单链表头哨兵；双链表用 0、1 为左右哨兵。插入返回稳定编号，删除不复用编号。O(1) 更新，删除须针对有效节点。

```rust
struct SingleList {
    value:Vec<i64>,next:Vec<Option<usize>>
}
impl SingleList {
    fn new()->Self{
        Self{
            value:vec![0],next:vec![None]
        }
    }
    fn insert_after(&mut self,k:usize,x:i64)->usize{
        let i=self.value.len();
        self.value.push(x);
        self.next.push(self.next[k]);
        self.next[k]=Some(i);
        i
    }
    fn remove_after(&mut self,k:usize){
        if let Some(i)=self.next[k]{
            self.next[k]=self.next[i];
        }
    }
}
struct DoubleList {
    value:Vec<i64>,left:Vec<usize>,right:Vec<usize>
}
impl DoubleList {
    fn new()->Self{
        Self{
            value:vec![0,0],left:vec![0,0],right:vec![1,1]
        }
    }
    fn insert_after(&mut self,k:usize,x:i64)->usize{
        let i=self.value.len();
        self.value.push(x);
        self.left.push(k);
        self.right.push(self.right[k]);
        let r=self.right[k];
        self.left[r]=i;
        self.right[k]=i;
        i
    }
    fn remove(&mut self,k:usize){
        let(l,r)=(self.left[k],self.right[k]);
        self.right[l]=r;
        self.left[r]=l;
    }
}
```

## 栈、队列、堆与标准集合

栈、队列与最小堆。队列避免从动态数组头部逐项删除；最小堆用于后文 Dijkstra、哈夫曼与区间分组。Go 的 PairHeap 使用标准 container/heap；TypeScript 的 Heap 为可传比较器的通用堆。

```rust
fn collections_example(){
    let mut stack=vec![1];
    stack.push(2);
    stack.pop();
    let mut q=VecDeque::from(vec![1,2]);
    q.pop_front();
    let mut heap=BinaryHeap::new();
    heap.push(Reverse(3i64));
    let _=heap.pop();
    let mut freq=HashMap::new();
    *freq.entry(3).or_insert(0)+=1;
    let mut seen=HashSet::new();
    seen.insert(3);
    let mut ordered=std::collections::BTreeSet::new();
    ordered.insert(3);
    let _=ordered.range(2..).next();
}
```

## 单调栈与单调队列

左侧最近严格较小下标（不存在 -1）与窗口最小值，O(n)。窗口须满足 1≤k≤n。

```rust
fn previous_less(a:&[i64])->Vec<isize>{
    let(mut st,mut res):(Vec<usize>,Vec<isize>)=(vec![],vec![]);
    for i in 0..a.len(){
        while st.last().map_or(false,|&j|a[j]>=a[i]){
            st.pop();
        }
        res.push(st.last().map_or(-1,|&j|j as isize));
        st.push(i);
    }
    res
}
fn window_min(a:&[i64],k:usize)->Vec<i64>{
    assert!(k>=1&&k<=a.len());
    let(mut q,mut res):(VecDeque<usize>,Vec<i64>)=(VecDeque::new(),vec![]);
    for i in 0..a.len(){
        while q.front().map_or(false,|&j|j+k<=i){
            q.pop_front();
        }
        while q.back().map_or(false,|&j|a[j]>=a[i]){
            q.pop_back();
        }
        q.push_back(i);
        if i+1>=k{
            res.push(a[*q.front().unwrap()]);
        }
    }
    res
}
```

## KMP 字符串匹配

返回全部匹配起点，支持重叠，空模式匹配每个边界。O(n+m)。Go/Rust 按 UTF-8 字节，TypeScript 按 UTF-16 码元编号。

```rust
fn kmp(text:&str,pattern:&str)->Vec<usize>{
    let(t,p)=(text.as_bytes(),pattern.as_bytes());
    if p.is_empty(){
        return (0..=t.len()).collect();
    }
    let mut pi=vec![0;
    p.len()];
    let mut j=0;
    for i in 1..p.len(){
        while j>0&&p[i]!=p[j]{
            j=pi[j-1];
        }
        if p[i]==p[j]{
            j+=1;
        }
        pi[i]=j;
    }
    let(mut res,mut j)=(vec![],0);
    for i in 0..t.len(){
        while j>0&&t[i]!=p[j]{
            j=pi[j-1];
        }
        if t[i]==p[j]{
            j+=1;
        }
        if j==p.len(){
            res.push(i+1-p.len());
            j=pi[j-1];
        }
    }
    res
}
```

## Trie 字典树

插入单词并统计完整单词出现次数，O(单词长度)。这里限定小写英文字母。

```rust
struct Trie {
    next:Vec<[Option<usize>;
    26]>,count:Vec<usize>
}
impl Trie {
    fn new()->Self{
        Self{
            next:vec![[None;
            26]],count:vec![0]
        }
    }
    fn insert(&mut self,s:&str){
        let mut p=0;
        for c in s.bytes(){
            let x=(c-b'a')as usize;
            let v=match self.next[p][x]{
                Some(v)=>v,None=>{
                    let v=self.next.len();
                    self.next.push([None;26]);
                    self.count.push(0);
                    self.next[p][x]=Some(v);
                    v
                }
            }             ;
            p=v;
        }
        self.count[p]+=1;
    }
    fn count(&self,s:&str)->usize{
        let mut p=0;
        for c in s.bytes(){
            match self.next[p][(c-b'a')as usize]{
                Some(v)=>p=v,None=>return 0
            }
        }
        self.count[p]
    }
}
```

## 并查集

路径压缩与按集合大小合并，均摊 O(α(n))。重复合并不改变大小。

```rust
struct DSU {
    parent:Vec<usize>,size:Vec<usize>
}
impl DSU {
    fn new(n:usize)->Self{
        Self{
            parent:(0..n).collect(),size:vec![1;
            n]
        }
    }
    fn find(&mut self,mut x:usize)->usize{
        while self.parent[x]!=x{
            self.parent[x]=self.parent[self.parent[x]];
            x=self.parent[x];
        }
        x
    }
    fn union(&mut self,a:usize,b:usize)->bool{
        let(mut a,mut b)=(self.find(a),self.find(b));
        if a==b{
            return false;
        }
        if self.size[a]<self.size[b]{
            std::mem::swap(&mut a,&mut b);
        }
        self.parent[b]=a;
        self.size[a]+=self.size[b];
        true
    }
}
```

## 可修改、可删除的编号堆

编号唯一的最小堆，支持插入、按编号修改和删除，O(log n)。peek 返回 {值,编号}；操作须针对存在的编号，peek 前须非空。

```rust
struct IndexedHeap {
    data:Vec<(i64,usize)>,pos:HashMap<usize,usize>
}
impl IndexedHeap {
    fn new()->Self{
        Self{
            data:vec![],pos:HashMap::new()
        }
    }
    fn swap(&mut self,i:usize,j:usize){
        self.data.swap(i,j);
        self.pos.insert(self.data[i].1,i);
        self.pos.insert(self.data[j].1,j);
    }
    fn up(&mut self,mut i:usize)->usize{
        while i>0{
            let p=(i-1)/2;
            if self.data[p].0<=self.data[i].0{
                break;
            }
            self.swap(i,p);
            i=p;
        }
        i
    }
    fn down(&mut self,mut i:usize){
        while 2*i+1<self.data.len(){
            let mut j=2*i+1;
            if j+1<self.data.len()&&self.data[j+1].0<self.data[j].0{
                j+=1;
            }
            if self.data[i].0<=self.data[j].0{
                break;
            }
            self.swap(i,j);
            i=j;
        }
    }
    fn insert(&mut self,id:usize,value:i64){
        assert!(!self.pos.contains_key(&id));
        self.pos.insert(id,self.data.len());
        self.data.push((value,id));
        self.up(self.data.len()-1);
    }
    fn change(&mut self,id:usize,value:i64){
        let i=self.pos[&id];
        self.data[i].0=value;
        let p=self.up(i);
        self.down(p);
    }
    fn remove(&mut self,id:usize){
        let i=self.pos[&id];
        let last=self.data.len()-1;
        self.swap(i,last);
        self.data.pop();
        self.pos.remove(&id);
        if i<self.data.len(){
            let p=self.up(i);
            self.down(p);
        }
    }
    fn peek(&self)->(i64,usize){
        self.data[0]
    }
}
```

## 哈希：开放寻址与链表法

开放寻址与链表法整数集合，仅插入和查询。容量为正，开放寻址容量须大于不同元素数。独立占用标记支持任意键值。

```rust
struct OpenHash {
    keys:Vec<Option<i64>>
}
impl OpenHash {
    fn new(n:usize)->Self{
        Self{
            keys:vec![None;
            n]
        }
    }
    fn locate(&self,x:i64)->usize{
        let mut k=x.rem_euclid(self.keys.len()as i64)as usize;
        let start=k;
        while self.keys[k].map_or(false,|v|v!=x){
            k=(k+1)%self.keys.len();
            assert!(k!=start,"full table");
        }
        k
    }
    fn add(&mut self,x:i64){
        let k=self.locate(x);
        self.keys[k]=Some(x);
    }
    fn contains(&self,x:i64)->bool{
        self.keys[self.locate(x)].is_some()
    }
}
struct ChainHash {
    buckets:Vec<Vec<i64>>
}
impl ChainHash {
    fn new(n:usize)->Self{
        Self{
            buckets:vec![vec![];
            n]
        }
    }
    fn bucket(&self,x:i64)->usize{
        x.rem_euclid(self.buckets.len()as i64)as usize
    }
    fn contains(&self,x:i64)->bool{
        self.buckets[self.bucket(x)].contains(&x)
    }
    fn add(&mut self,x:i64){
        if !self.contains(x){
            let k=self.bucket(x);
            self.buckets[k].push(x);
        }
    }
}
```

## 字符串前缀哈希

前缀哈希查询 [l,r)，O(n) 构建、O(1) 查询。base=131，mod=1000000007，有碰撞可能；精确比较需再核对原字符串。Go/Rust 按字节，TypeScript 按 UTF-16 码元。

```rust
struct StringHash {
    h:Vec<u64>,power:Vec<u64>
}
impl StringHash {
    fn new(s:&str)->Self{
        let(mut h,mut power)=(vec![0],vec![1]);
        for c in s.bytes(){
            h.push((h[h.len()-1]*131+c as u64+1)%1_000_000_007);
            power.push(power[power.len()-1]*131%1_000_000_007);
        }
        Self{
            h,power
        }
    }
    fn get(&self,l:usize,r:usize)->u64{
        (self.h[r]+1_000_000_007-self.h[l]*self.power[r-l]%1_000_000_007)%1_000_000_007
    }
}
```

## 二叉树遍历

一般二叉树的前、中、后序遍历，O(n)。Go/TypeScript 使用节点引用；Rust 使用节点数组与 Option<usize> 子节点索引。树须无环，迭代遍历。

```rust
struct TreeNode {
    value:i64,left:Option<usize>,right:Option<usize>
}
fn traversals(nodes:&[TreeNode],root:Option<usize>)->(Vec<i64>,Vec<i64>,Vec<i64>){
    let(mut pre,mut inorder,mut post)=(vec![],vec![],vec![]);
    let mut st:Vec<usize>=root.into_iter().collect();
    while let Some(u)=st.pop(){
        pre.push(nodes[u].value);
        if let Some(v)=nodes[u].right{
            st.push(v);
        }
        if let Some(v)=nodes[u].left{
            st.push(v);
        }
    }
    let mut cur=root;
    while cur.is_some()||!st.is_empty(){
        while let Some(u)=cur{
            st.push(u);
            cur=nodes[u].left;
        }
        let u=st.pop().unwrap();
        inorder.push(nodes[u].value);
        cur=nodes[u].right;
    }
    st.extend(root);
    while let Some(u)=st.pop(){
        post.push(nodes[u].value);
        if let Some(v)=nodes[u].left{
            st.push(v);
        }
        if let Some(v)=nodes[u].right{
            st.push(v);
        }
    }
    post.reverse();
    (pre,inorder,post)
}
```

## 哈夫曼树与合并代价

非负权值的最小合并代价，即哈夫曼带权路径长度。空集合或单节点为 0，O(n log n)。Go/TypeScript 依赖前文 PairHeap / Heap。

```rust
fn huffman_cost(a:&[i64])->i64{
    let mut h:BinaryHeap<Reverse<i64>>=a.iter().map(|&x|Reverse(x)).collect();
    let mut ans=0;
    while h.len()>1{
        let s=h.pop().unwrap().0+h.pop().unwrap().0;
        ans+=s;
        h.push(Reverse(s));
    }
    ans
}
```

## 树状数组：点加、区间加与区间和

本节一基下标。点加、前缀和与区间和；双树状数组支持闭区间 [l,r] 加与区间和。初始为零，O(log n)。

```rust
struct Fenwick {
    tree:Vec<i64>
}
impl Fenwick {
    fn new(n:usize)->Self{
        Self{
            tree:vec![0;
            n+1]
        }
    }
    fn add(&mut self,mut i:usize,v:i64){
        assert!(i>0);
        while i<self.tree.len(){
            self.tree[i]+=v;
            i+=i&i.wrapping_neg();
        }
    }
    fn prefix(&self,mut i:usize)->i64{
        let mut s=0;
        while i>0{
            s+=self.tree[i];
            i-=i&i.wrapping_neg();
        }
        s
    }
    fn range_sum(&self,l:usize,r:usize)->i64{
        self.prefix(r)-self.prefix(l-1)
    }
}
struct RangeFenwick {
    b1:Fenwick,b2:Fenwick
}
impl RangeFenwick {
    fn new(n:usize)->Self{
        Self{
            b1:Fenwick::new(n),b2:Fenwick::new(n)
        }
    }
    fn range_add(&mut self,l:usize,r:usize,v:i64){
        self.b1.add(l,v);
        self.b1.add(r+1,-v);
        self.b2.add(l,v*(l as i64-1));
        self.b2.add(r+1,-v*r as i64);
    }
    fn prefix(&self,i:usize)->i64{
        self.b1.prefix(i)*i as i64-self.b2.prefix(i)
    }
    fn range_sum(&self,l:usize,r:usize)->i64{
        self.prefix(r)-self.prefix(l-1)
    }
}
```

## 线段树：懒标记区间加、区间和

非空数组，零基闭区间 [l,r]。懒标记支持区间加与区间和，O(log n)。

```rust
struct SegmentTree {
    n:usize,sum:Vec<i64>,lazy:Vec<i64>
}
impl SegmentTree {
    fn new(a:&[i64])->Self{
        assert!(!a.is_empty());
        let mut s=Self{
            n:a.len(),sum:vec![0;
            4*a.len()],lazy:vec![0;
            4*a.len()]
        }         ;
        s.build(1,0,a.len()-1,a);
        s
    }
    fn build(&mut self,u:usize,l:usize,r:usize,a:&[i64]){
        if l==r{
            self.sum[u]=a[l];
            return;
        }
        let m=(l+r)/2;
        self.build(u*2,l,m,a);
        self.build(u*2+1,m+1,r,a);
        self.pull(u);
    }
    fn pull(&mut self,u:usize){
        self.sum[u]=self.sum[u*2]+self.sum[u*2+1];
    }
    fn apply(&mut self,u:usize,l:usize,r:usize,v:i64){
        self.sum[u]+=(r-l+1)as i64*v;
        self.lazy[u]+=v;
    }
    fn push(&mut self,u:usize,l:usize,r:usize){
        let m=(l+r)/2;
        let v=self.lazy[u];
        self.apply(u*2,l,m,v);
        self.apply(u*2+1,m+1,r,v);
        self.lazy[u]=0;
    }
    fn add(&mut self,ql:usize,qr:usize,v:i64){
        self.update(1,0,self.n-1,ql,qr,v);
    }
    fn update(&mut self,u:usize,l:usize,r:usize,ql:usize,qr:usize,v:i64){
        if ql<=l&&r<=qr{
            self.apply(u,l,r,v);
            return;
        }
        self.push(u,l,r);
        let m=(l+r)/2;
        if ql<=m{
            self.update(u*2,l,m,ql,qr,v);
        }
        if qr>m{
            self.update(u*2+1,m+1,r,ql,qr,v);
        }
        self.pull(u);
    }
    fn query(&mut self,ql:usize,qr:usize)->i64{
        self.get(1,0,self.n-1,ql,qr)
    }
    fn get(&mut self,u:usize,l:usize,r:usize,ql:usize,qr:usize)->i64{
        if ql<=l&&r<=qr{
            return self.sum[u];
        }
        self.push(u,l,r);
        let m=(l+r)/2;
        let mut ans=0;
        if ql<=m{
            ans+=self.get(u*2,l,m,ql,qr);
        }
        if qr>m{
            ans+=self.get(u*2+1,m+1,r,ql,qr);
        }
        ans
    }
}
```

## 树形 DP：最大权独立集

非空无向树，最大权独立集：相邻节点不能同时选，可全不选。迭代建树后逆序转移，O(n)。

```rust
fn tree_independent_set(g:&[Vec<usize>],w:&[i64])->i64{
    let n=g.len();
    let mut p=vec![usize::MAX;
    n];
    p[0]=0;
    let mut order=vec![0];
    let mut i=0;
    while i<order.len(){
        let u=order[i];
        for &v in &g[u]{
            if p[v]==usize::MAX{
                p[v]=u;
                order.push(v);
            }
        }
        i+=1;
    }
    let mut f=vec![[0i64;
    2];
    n];
    for &u in order.iter().rev(){
        f[u][1]+=w[u];
        if u!=0{
            f[p[u]][0]+=f[u][0].max(f[u][1]);
            f[p[u]][1]+=f[u][0];
        }
    }
    f[0][0].max(f[0][1])
}
```

# 搜索与图论

## DFS：全排列与 N 皇后

返回全排列与 N 皇后的全部解；内存随解数增长，仅用于搜索深度较小的题。皇后解记录各行列号。

```rust
fn permutations(n:usize)->Vec<Vec<usize>>{
    fn dfs(path:&mut Vec<usize>,used:&mut [bool],ans:&mut Vec<Vec<usize>>){
        if path.len()==used.len(){
            ans.push(path.clone());
            return;
        }
        for x in 0..used.len(){
            if !used[x]{
                used[x]=true;
                path.push(x);
                dfs(path,used,ans);
                path.pop();
                used[x]=false;
            }
        }
    }
    let mut ans=vec![];
    dfs(&mut vec![],&mut vec![false;n],&mut ans);
    ans
}
fn n_queens(n:usize)->Vec<Vec<usize>>{
    fn dfs(n:usize,path:&mut Vec<usize>,col:&mut [bool],d1:&mut [bool],d2:&mut [bool],ans:&mut Vec<Vec<usize>>){
        let r=path.len();
        if r==n{
            ans.push(path.clone());
            return;
        }
        for c in 0..n{
            let d=r+n-c;
            if !col[c]&&!d1[r+c]&&!d2[d]{
                col[c]=true;
                d1[r+c]=true;
                d2[d]=true;
                path.push(c);
                dfs(n,path,col,d1,d2,ans);
                path.pop();
                col[c]=false;
                d1[r+c]=false;
                d2[d]=false;
            }
        }
    }
    let mut ans=vec![];
    dfs(n,&mut vec![],&mut vec![false;n],&mut vec![false;2*n+1],&mut vec![false;2*n+1],&mut ans);
    ans
}
```

## 树的重心

树的重心指标：删除单个节点后最大连通块大小的最小值；非空无向树，O(n)。

```rust
fn centroid_balance(g:&[Vec<usize>])->usize{
    let n=g.len();
    let(mut p,mut size,mut order)=(vec![usize::MAX;n],vec![1;n],vec![0]);
    p[0]=0;
    let mut i=0;
    while i<order.len(){
        let u=order[i];
        for &v in &g[u]{
            if p[v]==usize::MAX{
                p[v]=u;
                order.push(v);
            }
        }
        i+=1;
    }
    let mut best=n;
    for &u in order.iter().rev(){
        let mut largest=0;
        for &v in &g[u]{
            if v!=u&&p[v]==u{
                size[u]+=size[v];
                largest=largest.max(size[v]);
            }
        }
        best=best.min(largest.max(n-size[u]));
    }
    best
}
```

## BFS：图的层次与迷宫

无权图距离，不可达 -1；迷宫 0 可走、1 障碍，起终点为左上与右下。O(V+E) / O(nm)。

```rust
fn bfs(g:&[Vec<usize>],source:usize)->Vec<i64>{
    let mut d=vec![-1;
    g.len()];
    d[source]=0;
    let mut q=VecDeque::from(vec![source]);
    while let Some(u)=q.pop_front(){
        for &v in &g[u]{
            if d[v]==-1{
                d[v]=d[u]+1;
                q.push_back(v);
            }
        }
    }
    d
}
fn maze_distance(a:&[Vec<i32>])->i64{
    if a.is_empty()||a[0].is_empty(){
        return -1;
    }
    let(n,m)=(a.len(),a[0].len());
    if a[0][0]!=0||a[n-1][m-1]!=0{
        return -1;
    }
    let mut d=vec![vec![-1;
    m];
    n];
    d[0][0]=0;
    let mut q=VecDeque::from(vec![(0,0)]);
    while let Some((x,y))=q.pop_front(){
        for(dx,dy)in[(-1,0),(0,1),(1,0),(0,-1)]{
            let(nx,ny)=(x as isize+dx,y as isize+dy);
            if nx>=0&&ny>=0&&(nx as usize)<n&&(ny as usize)<m{
                let(nx,ny)=(nx as usize,ny as usize);
                if a[nx][ny]==0&&d[nx][ny]==-1{
                    d[nx][ny]=d[x][y]+1;
                    q.push_back((nx,ny));
                }
            }
        }
    }
    d[n-1][m-1]
}
```

## Dijkstra：朴素与堆优化

边权非负。矩阵无边用 INF（Go/Rust 的最大 i64 /4）或 Infinity；邻接项为 {终点,权重}。朴素 O(n²)，堆优化 O((V+E) log V)。Go/TypeScript 依赖 PairHeap / Heap。

```rust
const INF:i64=i64::MAX/4;
type Edge=(usize,i64);
fn dijkstra_dense(g:&[Vec<i64>],s:usize)->Vec<i64>{
    let n=g.len();
    let(mut d,mut used)=(vec![INF;n],vec![false;n]);
    d[s]=0;
    for _ in 0..n{
        let u=(0..n).filter(|&v|!used[v]).min_by_key(|&v|d[v]);
        let u=match u{
            Some(u)if d[u]!=INF=>u,_=>break
        }         ;
        used[u]=true;
        for v in 0..n{
            if g[u][v]!=INF{
                d[v]=d[v].min(d[u]+g[u][v]);
            }
        }
    }
    d
}
fn dijkstra(g:&[Vec<Edge>],s:usize)->Vec<i64>{
    let mut d=vec![INF;
    g.len()];
    d[s]=0;
    let mut h=BinaryHeap::from(vec![Reverse((0,s))]);
    while let Some(Reverse((dist,u)))=h.pop(){
        if dist!=d[u]{
            continue;
        }
        for &(v,w)in &g[u]{
            if d[v]>dist+w{
                d[v]=dist+w;
                h.push(Reverse((d[v],v)));
            }
        }
    }
    d
}
```

## Bellman-Ford：最多 k 条边

边表 {u,v,w}，允许负权；最多 k 条边，每轮使用上一轮副本。不可达 INF / Infinity，O(kE)。Go/Rust 依赖前文 INF。

```rust
type Arc=(usize,usize,i64);
fn bellman_ford(n:usize,edges:&[Arc],s:usize,k:usize)->Vec<i64>{
    let mut d=vec![INF;
    n];
    d[s]=0;
    for _ in 0..k{
        let old=d.clone();
        for &(u,v,w)in edges{
            if old[u]!=INF{
                d[v]=d[v].min(old[u]+w);
            }
        }
    }
    d
}
```

## SPFA

允许负权；源点可达负环时 Go 返回 false、TypeScript 抛异常、Rust 返回 None。不可达 INF / Infinity，最坏 O(VE)。依赖 Edge 与 INF（TypeScript 不需要 INF）。

```rust
fn spfa(g:&[Vec<Edge>],s:usize)->Option<Vec<i64>>{
    let n=g.len();
    let(mut d,mut count,mut queued)=(vec![INF;n],vec![0;n],vec![false;n]);
    d[s]=0;
    queued[s]=true;
    let mut q=VecDeque::from(vec![s]);
    while let Some(u)=q.pop_front(){
        queued[u]=false;
        for &(v,w)in &g[u]{
            if d[v]>d[u]+w{
                d[v]=d[u]+w;
                count[v]=count[u]+1;
                if count[v]>=n{
                    return None;
                }
                if !queued[v]{
                    queued[v]=true;
                    q.push_back(v);
                }
            }
        }
    }
    Some(d)
}
```

## Floyd 全源最短路

无负环；对角线 0，无边 INF / Infinity，重边建图取最小值。返回副本，O(n³)。Go/Rust 依赖 INF。

```rust
fn floyd(g:&[Vec<i64>])->Vec<Vec<i64>>{
    let mut d=g.to_vec();
    for k in 0..d.len(){
        for i in 0..d.len(){
            if d[i][k]!=INF{
                for j in 0..d.len(){
                    if d[k][j]!=INF{
                        d[i][j]=d[i][j].min(d[i][k]+d[k][j]);
                    }
                }
            }
        }
    }
    d
}
```

## 最小生成树：Prim 与 Kruskal

无向图，允许负权。Prim 使用矩阵（无边 INF / Infinity），O(n²)；Kruskal 边表使用 Arc，O(E log E)，依赖 DSU。非连通为 Go false、TS null、Rust None，空图代价 0。

```rust
fn prim(g:&[Vec<i64>])->Option<i64>{
    let n=g.len();
    if n==0{
        return Some(0);
    }
    let(mut d,mut used,mut ans)=(vec![INF;n],vec![false;n],0);
    d[0]=0;
    for _ in 0..n{
        let u=(0..n).filter(|&v|!used[v]).min_by_key(|&v|d[v]).unwrap();
        if d[u]==INF{
            return None;
        }
        ans+=d[u];
        used[u]=true;
        for v in 0..n{
            if !used[v]{
                d[v]=d[v].min(g[u][v]);
            }
        }
    }
    Some(ans)
}
fn kruskal(n:usize,edges:&[Arc])->Option<i64>{
    let mut a=edges.to_vec();
    a.sort_unstable_by_key(|e|e.2);
    let(mut d,mut ans,mut count)=(DSU::new(n),0,0);
    for(u,v,w)in a{
        if d.union(u,v){
            ans+=w;
            count+=1;
        }
    }
    if n==0||count+1==n{
        Some(ans)
    }
    else{
        None
    }
}
```

## 二分图：染色与匈牙利算法

无向图染色 O(V+E)。最大匹配输入仅左点邻接表，邻接值为右点编号；迭代增广 O(VE)，不受递归栈深度限制。

```rust
fn is_bipartite(g:&[Vec<usize>])->bool{
    let mut color=vec![0;
    g.len()];
    for s in 0..g.len(){
        if color[s]!=0{
            continue;
        }
        color[s]=1;
        let mut q=VecDeque::from(vec![s]);
        while let Some(u)=q.pop_front(){
            for &v in &g[u]{
                if color[v]==color[u]{
                    return false;
                }
                if color[v]==0{
                    color[v]=3-color[u];
                    q.push_back(v);
                }
            }
        }
    }
    true
}
fn max_matching(g:&[Vec<usize>],nr:usize)->usize{
    let(mut left,mut right):(Vec<Option<usize>>,Vec<Option<usize>>)=(vec![None;g.len()],vec![None;nr]);
    let mut ans=0;
    for s in 0..g.len(){
        let(mut prev,mut seen)=(vec![None;nr],vec![false;g.len()]);
        seen[s]=true;
        let mut q=VecDeque::from(vec![s]);
        let mut end=None;
        while end.is_none(){
            let u=match q.pop_front(){
                Some(u)=>u,None=>break
            }             ;
            for &v in &g[u]{
                if prev[v].is_some(){
                    continue;
                }
                prev[v]=Some(u);
                if let Some(next)=right[v]{
                    if !seen[next]{
                        seen[next]=true;
                        q.push_back(next);
                    }
                }
                else{
                    end=Some(v);
                    break;
                }
            }
        }
        if end.is_some(){
            ans+=1;
            while let Some(v)=end{
                let u=prev[v].unwrap();
                end=left[u];
                left[u]=Some(v);
                right[v]=Some(u);
            }
        }
    }
    ans
}
```

## 拓扑排序

有向图拓扑排序，O(V+E)。存在环时 Go 返回 false、TypeScript null、Rust None。

```rust
fn topological_sort(g:&[Vec<usize>])->Option<Vec<usize>>{
    let mut deg=vec![0;
    g.len()];
    for row in g{
        for &v in row{
            deg[v]+=1;
        }
    }
    let mut q:Vec<usize>=(0..g.len()).filter(|&v|deg[v]==0).collect();
    let mut h=0;
    while h<q.len(){
        for &v in &g[q[h]]{
            deg[v]-=1;
            if deg[v]==0{
                q.push(v);
            }
        }
        h+=1;
    }
    if q.len()==g.len(){
        Some(q)
    }
    else{
        None
    }
}
```

# 数学知识

## 质数：试除、分解与筛法

试除 O(√n)，分解返回质因子指数表（n≥1）。两种筛均包含上界 n：埃氏 O(n log log n)、线性 O(n)。

```rust
fn is_prime(n:u64)->bool{
    if n<2{
        return false;
    }
    let mut p=2;
    while p<=n/p{
        if n%p==0{
            return false;
        }
        p+=1;
    }
    true
}
fn factorize(mut n:u64)->HashMap<u64,u32>{
    assert!(n>=1);
    let(mut f,mut p)=(HashMap::new(),2);
    while p<=n/p{
        while n%p==0{
            *f.entry(p).or_insert(0)+=1;
            n/=p;
        }
        p+=1;
    }
    if n>1{
        *f.entry(n).or_insert(0)+=1;
    }
    f
}
fn sieve(n:usize)->Vec<usize>{
    let(mut comp,mut primes)=(vec![false;n+1],vec![]);
    for i in 2..=n{
        if !comp[i]{
            primes.push(i);
            if i<=n/i{
                for j in (i*i..=n).step_by(i){
                    comp[j]=true;
                }
            }
        }
    }
    primes
}
fn linear_sieve(n:usize)->Vec<usize>{
    let(mut comp,mut primes)=(vec![false;n+1],vec![]);
    for i in 2..=n{
        if !comp[i]{
            primes.push(i);
        }
        for &p in &primes{
            if p>n/i{
                break;
            }
            comp[i*p]=true;
            if i%p==0{
                break;
            }
        }
    }
    primes
}
```

## 约数、约数个数与约数之和

枚举正整数约数并排序；约数个数与和从质因子指数表计算。模数须正；Go 限定 mod≤2147483647，Rust 模数用 u32 并用 u128 中间乘法，TypeScript 用 bigint。

```rust
fn divisors(n:u64)->Vec<u64>{
    assert!(n>=1);
    let(mut a,mut i)=(vec![],1);
    while i<=n/i{
        if n%i==0{
            a.push(i);
            if i!=n/i{
                a.push(n/i);
            }
        }
        i+=1;
    }
    a.sort_unstable();
    a
}
fn divisor_count(f:&HashMap<u64,u32>,modulus:u32)->u64{
    let m=modulus as u64;
    let mut ans=1%m;
    for &e in f.values(){
        ans=ans*(e as u64+1)%m;
    }
    ans
}
fn divisor_sum(f:&HashMap<u64,u32>,modulus:u32)->u64{
    let m=modulus as u128;
    let mut ans=1%m;
    for(&p,&e)in f{
        let mut term=1%m;
        for _ in 0..e{
            term=(term*p as u128+1)%m;
        }
        ans=ans*term%m;
    }
    ans as u64
}
```

## 欧几里得、扩展欧几里得与线性同余方程

a、b 非负且不同时为 0；扩展欧几里得返回 (g,x,y)，ax+by=g。线性同余方程 ax≡b(mod m)，m>0；无解为 false / null / None。Go/Rust 系数中间值须在 i64 范围内，TS 使用 bigint。

```rust
fn gcd(mut a:i64,mut b:i64)->i64{
    while b!=0{
        let r=a%b;
        a=b;
        b=r;
    }
    a
}
fn exgcd(a:i64,b:i64)->(i64,i64,i64){
    if b==0{
        return(a,1,0);
    }
    let(g,x,y)=exgcd(b,a%b);
    (g,y,x-a/b*y)
}
fn solve_congruence(a:i64,b:i64,m:i64)->Option<i64>{
    let(g,x,_)=exgcd(a,m);
    if b%g!=0{
        None
    }
    else{
        Some((x*(b/g)).rem_euclid(m/g))
    }
}
```

## 快速幂

指数非负，模数正，O(log b)。Go 模数≤2147483647；Rust 模数 u32，乘法用 u64；TypeScript 使用 bigint。

```rust
fn mod_pow(a:u64,mut b:u64,modulus:u32)->u64{
    let m=modulus as u64;
    let(mut a,mut res)=(a%m,1%m);
    while b>0{
        if b&1!=0{
            res=res*a%m;
        }
        a=a*a%m;
        b>>=1;
    }
    res
}
```

## 高斯消元

n×(n+1) 增广矩阵，直接修改；0=唯一解、1=无穷多解、2=无解。唯一解存最后一列，O(n³)，eps=1e-9。

```rust
fn gauss(a:&mut [Vec<f64>])->u8{
    let n=a.len();
    let mut r=0;
    let eps=1e-9;
    for c in 0..n{
        let mut t=r;
        for i in r..n{
            if a[i][c].abs()>a[t][c].abs(){
                t=i;
            }
        }
        if a[t][c].abs()<eps{
            continue;
        }
        a.swap(t,r);
        for j in(c..=n).rev(){
            a[r][j]/=a[r][c];
        }
        for i in r+1..n{
            if a[i][c].abs()>eps{
                for j in(c..=n).rev(){
                    a[i][j]-=a[r][j]*a[i][c];
                }
            }
        }
        r+=1;
    }
    if r<n{
        for row in &a[r..]{
            if row[n].abs()>eps{
                return 2;
            }
        }
        return 1;
    }
    for i in(0..n).rev(){
        for j in i+1..n{
            a[i][n]-=a[i][j]*a[j][n];
        }
    }
    0
}
```

## 组合数：递推、阶乘逆元、Lucas 与精确值

递推适用任意正模数。阶乘逆元要求 p 为质数、n<p；Lucas 要求 p 为质数且适合较小 p，小组合 O(p)。依赖 ModPow / modPow / mod_pow。Go 模数≤2147483647；Rust 用 u32；精确组合数 Go 用 math/big、TS 用 bigint，Rust 返回十进制字符串并依赖后文数位运算。

```rust
fn combination_table(n:usize,modulus:u32)->Vec<Vec<u64>>{
    let m=modulus as u64;
    let mut c=vec![vec![0;
    n+1];
    n+1];
    for i in 0..=n{
        c[i][0]=1%m;
        for j in 1..=i{
            c[i][j]=(c[i-1][j-1]+c[i-1][j])%m;
        }
    }
    c
}
struct CombPrime {
    fact:Vec<u64>,inv:Vec<u64>,p:u32
}
impl CombPrime {
    fn new(n:usize,p:u32)->Self{
        assert!(p>=2&&n<(p as usize));
        let m=p as u64;
        let(mut fact,mut inv)=(vec![1;n+1],vec![1;n+1]);
        for i in 1..=n{
            fact[i]=fact[i-1]*i as u64%m;
        }
        inv[n]=mod_pow(fact[n],(p-2)as u64,p);
        for i in(1..=n).rev(){
            inv[i-1]=inv[i]*i as u64%m;
        }
        Self{
            fact,inv,p
        }
    }
    fn choose(&self,n:usize,k:usize)->u64{
        if k>n{
            0
        }
        else{
            self.fact[n]*self.inv[k]%self.p as u64*self.inv[n-k]%self.p as u64
        }
    }
}
fn small_comb(n:u64,k:u64,p:u32)->u64{
    if k>n{
        return 0;
    }
    let(mut a,mut b)=(1,1);
    for i in 1..=k.min(n-k){
        a=a*(n-i+1)%p as u64;
        b=b*i%p as u64;
    }
    a*mod_pow(b,(p-2)as u64,p)%p as u64
}
fn lucas(mut n:u64,mut k:u64,p:u32)->u64{
    assert!(p>=2);
    let m=p as u64;
    let mut ans=1;
    while n>0||k>0{
        ans=ans*small_comb(n%m,k%m,p)%m;
        n/=m;
        k/=m;
    }
    ans
}
fn exact_comb(n:u32,k:u32)->String{
    if k>n{
        return "0".into();
    }
    let mut a=vec![1];
    for i in 1..=k.min(n-k){
        a=multiply_digits(&a,(n-i+1)as u64);
        a=divide_digits(&a,i as u64).0;
    }
    digits_string(&a)
}
```

## 卡特兰数

返回精确卡特兰数 C(2n,n)/(n+1)，n≥0。依赖精确组合数；Rust 还依赖后文数位运算，n 须使 2n 可用 u32 表示。

```rust
fn catalan(n:u32)->String{
    let a=digits(&exact_comb(n.checked_mul(2).unwrap(),n));
    digits_string(&divide_digits(&a,n as u64+1).0)
}
```

# 贪心

## 合并果子、区间覆盖、分组与选点

合并果子调用前文哈夫曼代价。闭区间覆盖要求 start<end，失败返回 -1；区间分组端点相接冲突；选点可用公共端点覆盖多段。O(n log n)，Go/TS 分组依赖最小堆。

```rust
fn cover_intervals(mut start:i64,end:i64,ranges:&[(i64,i64)])->i64{
    let mut a=ranges.to_vec();
    a.sort_unstable();
    let(mut i,mut count)=(0,0);
    while start<end{
        let mut next=start;
        while i<a.len()&&a[i].0<=start{
            next=next.max(a[i].1);
            i+=1;
        }
        if next==start{
            return -1;
        }
        start=next;
        count+=1;
    }
    count
}
fn group_intervals(ranges:&[(i64,i64)])->usize{
    let mut a=ranges.to_vec();
    a.sort_unstable();
    let mut h:BinaryHeap<Reverse<i64>>=BinaryHeap::new();
    for(l,r)in a{
        if h.peek().map_or(false,|x|x.0<l){
            h.pop();
        }
        h.push(Reverse(r));
    }
    h.len()
}
fn select_points(ranges:&[(i64,i64)])->usize{
    let mut a=ranges.to_vec();
    a.sort_unstable_by_key(|x|x.1);
    let(mut point,mut count)=(None,0);
    for(l,r)in a{
        if point.map_or(true,|x|x<l){
            point=Some(r);
            count+=1;
        }
    }
    count
}
```

# 动态规划

## 背包：01、完全、多重与分组

容量至多 m，可不选；体积为正、数量非负，价值可负。01/完全 O(nm)，多重二进制拆分 O(mΣlog(s+1))，分组 O(mΣ组大小)。

```rust
type Item=(usize,i64);
type BoundedItem=(usize,i64,usize);
fn knapsack_01(m:usize,items:&[Item])->i64{
    let mut f=vec![0;
    m+1];
    for &(v,w)in items{
        for j in(v..=m).rev(){
            f[j]=f[j].max(f[j-v]+w);
        }
    }
    f[m]
}
fn knapsack_complete(m:usize,items:&[Item])->i64{
    let mut f=vec![0;
    m+1];
    for &(v,w)in items{
        for j in v..=m{
            f[j]=f[j].max(f[j-v]+w);
        }
    }
    f[m]
}
fn knapsack_bounded(m:usize,items:&[BoundedItem])->i64{
    let mut f=vec![0;
    m+1];
    for &(v,w,s)in items{
        let(mut remaining,mut k)=(s.min(m/v),1usize);
        while remaining>0{
            let take=k.min(remaining);
            remaining-=take;
            let(volume,value)=(v*take,w*take as i64);
            for j in(volume..=m).rev(){
                f[j]=f[j].max(f[j-volume]+value);
            }
            if remaining==0{
                break;
            }
            k=k.saturating_mul(2).min(remaining);
        }
    }
    f[m]
}
fn knapsack_grouped(m:usize,groups:&[Vec<Item>])->i64{
    let mut f=vec![0;
    m+1];
    for group in groups{
        let old=f.clone();
        for j in 0..=m{
            for &(v,w)in group{
                if j>=v{
                    f[j]=f[j].max(old[j-v]+w);
                }
            }
        }
    }
    f[m]
}
```

## 线性 DP：数字三角形

三角形第 i 行 i+1 项，最大路径和，可负数；自底向上 O(n²) 时间、O(n) 额外空间。空输入返回 0。

```rust
fn triangle_max(a:&[Vec<i64>])->i64{
    if a.is_empty(){
        return 0;
    }
    let mut f=a.last().unwrap().clone();
    for i in(0..a.len()-1).rev(){
        for j in 0..=i{
            f[j]=a[i][j]+f[j].max(f[j+1]);
        }
    }
    f[0]
}
```

## 最长上升子序列

严格上升，O(n²) 返回一条正向序列；O(n log n) 版本只返回长度。空输入安全。

```rust
fn lis(a:&[i64])->Vec<i64>{
    if a.is_empty(){
        return vec![];
    }
    let(mut f,mut p,mut best)=(vec![1;a.len()],vec![None;a.len()],0);
    for i in 0..a.len(){
        for j in 0..i{
            if a[j]<a[i]&&f[j]+1>f[i]{
                f[i]=f[j]+1;
                p[i]=Some(j);
            }
        }
        if f[i]>f[best]{
            best=i;
        }
    }
    let(mut ans,mut cur)=(vec![],Some(best));
    while let Some(i)=cur{
        ans.push(a[i]);
        cur=p[i];
    }
    ans.reverse();
    ans
}
fn lis_length(a:&[i64])->usize{
    let mut tails=vec![];
    for &x in a{
        let i=lower_bound(&tails,x);
        if i==tails.len(){
            tails.push(x);
        }
        else{
            tails[i]=x;
        }
    }
    tails.len()
}
```

## 最长公共子序列

最长公共子序列长度，O(nm) 时间、O(m) 空间。Go/Rust 按字节，TypeScript 按 UTF-16 码元比较。

```rust
fn lcs_length(a:&str,b:&str)->usize{
    let(a,b)=(a.as_bytes(),b.as_bytes());
    let mut f=vec![0;
    b.len()+1];
    for &x in a{
        let mut diagonal=0;
        for j in 1..=b.len(){
            let old=f[j];
            f[j]=if x==b[j-1]{
                diagonal+1
            }
            else{
                f[j].max(f[j-1])
            }             ;
            diagonal=old;
        }
    }
    f[b.len()]
}
```

## 区间 DP：石子合并

线性排列的石子，每次合并相邻两堆，求最小总代价。O(n³) 时间、O(n²) 空间。

```rust
fn stone_merge(a:&[i64])->i64{
    let n=a.len();
    if n==0{
        return 0;
    }
    let s=prefix(a);
    let mut f=vec![vec![0;
    n];
    n];
    for len in 2..=n{
        for l in 0..=n-len{
            let r=l+len-1;
            f[l][r]=INF;
            for k in l..r{
                f[l][r]=f[l][r].min(f[l][k]+f[k+1][r]+s[r+1]-s[l]);
            }
        }
    }
    f[0][n-1]
}
```

# 高精度

## 大整数：加、减、乘、除

十进制非负整数的加、减、乘、除；减法允许负结果，除数必须正。Go 使用 math/big；TS 使用 bigint；Rust 依赖下一节数位运算，不需外部 crate。除法返回商与余数，均为字符串。

```rust
fn big_add(a:&str,b:&str)->String{
    digits_string(&add_digits(&digits(a),&digits(b)))
}
fn big_subtract(a:&str,b:&str)->String{
    let(a,b)=(digits(a),digits(b));
    if compare_digits(&a,&b)==std::cmp::Ordering::Less{
        format!("-{}",digits_string(&subtract_digits(&b,&a)))
    }
    else{
        digits_string(&subtract_digits(&a,&b))
    }
}
fn big_multiply(a:&str,b:&str)->String{
    let(a,b)=(digits(a),digits(b));
    let mut c=vec![0u32;
    a.len()+b.len()+1];
    for i in 0..a.len(){
        let mut carry=0;
        for j in 0..b.len(){
            let v=c[i+j]+a[i]*b[j]+carry;
            c[i+j]=v%10;
            carry=v/10;
        }
        let mut j=i+b.len();
        while carry>0{
            let v=c[j]+carry;
            c[j]=v%10;
            carry=v/10;
            j+=1;
        }
    }
    trim_digits(&mut c);
    digits_string(&c)
}
fn big_divide(a:&str,b:&str)->(String,String){
    let(a,b)=(digits(a),digits(b));
    assert!(b!=vec![0]);
    let(mut q,mut r)=(vec![],vec![0]);
    for &x in a.iter().rev(){
        r.insert(0,x);
        trim_digits(&mut r);
        let mut count=0;
        while compare_digits(&r,&b)!=std::cmp::Ordering::Less{
            r=subtract_digits(&r,&b);
            count+=1;
        }
        q.push(count);
    }
    q.reverse();
    trim_digits(&mut q);
    (digits_string(&q),digits_string(&r))
}
```

## 手写高精度：十进制数位数组

数位低位在前，至少一位，无多余高位零。加法、非负减法（A≥B）、乘非负小整数、除正小整数。Go/TS 小整数≤10^9；Go 中间值 int64、TS 中间值仍在安全整数内。Rust 小整数 u64，中间值 u128。

```rust
fn digits(s:&str)->Vec<u32>{
    assert!(!s.is_empty()&&s.bytes().all(|c|c.is_ascii_digit()));
    let mut a=s.bytes().rev().map(|c|(c-b'0')as u32).collect();
    trim_digits(&mut a);
    a
}
fn trim_digits(a:&mut Vec<u32>){
    while a.len()>1&&a.last()==Some(&0){
        a.pop();
    }
}
fn digits_string(a:&[u32])->String{
    a.iter().rev().map(|&x|(b'0'+x as u8)as char).collect()
}
fn compare_digits(a:&[u32],b:&[u32])->std::cmp::Ordering{
    a.len().cmp(&b.len()).then_with(||a.iter().rev().cmp(b.iter().rev()))
}
fn add_digits(a:&[u32],b:&[u32])->Vec<u32>{
    let(mut c,mut carry,mut i)=(vec![],0,0);
    while i<a.len().max(b.len())||carry>0{
        carry+=a.get(i).copied().unwrap_or(0)+b.get(i).copied().unwrap_or(0);
        c.push(carry%10);
        carry/=10;
        i+=1;
    }
    c
}
fn subtract_digits(a:&[u32],b:&[u32])->Vec<u32>{
    let(mut c,mut borrow)=(vec![],0i32);
    for i in 0..a.len(){
        let mut x=a[i]as i32-borrow-b.get(i).copied().unwrap_or(0)as i32;
        borrow=if x<0{
            1
        }
        else{
            0
        }         ;
        if x<0{
            x+=10;
        }
        c.push(x as u32);
    }
    trim_digits(&mut c);
    c
}
fn multiply_digits(a:&[u32],b:u64)->Vec<u32>{
    let(mut c,mut carry,mut i)=(vec![],0u128,0);
    while i<a.len()||carry>0{
        if i<a.len(){
            carry+=a[i]as u128*b as u128;
        }
        c.push((carry%10)as u32);
        carry/=10;
        i+=1;
    }
    trim_digits(&mut c);
    c
}
fn divide_digits(a:&[u32],b:u64)->(Vec<u32>,u64){
    assert!(b>0);
    let(mut c,mut r)=(vec![0;a.len()],0u128);
    for i in(0..a.len()).rev(){
        r=r*10+a[i]as u128;
        c[i]=(r/b as u128)as u32;
        r%=b as u128;
    }
    trim_digits(&mut c);
    (c,r as u64)
}
```

# 莫队

## 普通莫队：区间不同元素数

普通莫队，区间不同元素数；零基闭区间 [l,r]，原查询顺序返回。内部窗口使用 [L,R)；右端点奇偶块排序，典型 O((n+q)√n)。

```rust
fn mo_adjust(f:&mut HashMap<i64,usize>,x:i64,delta:i32){
    if delta>0{
        *f.entry(x).or_insert(0)+=1;
    }
    else{
        let count=f.get_mut(&x).unwrap();
        *count-=1;
        if *count==0{
            f.remove(&x);
        }
    }
}
fn mo_distinct(a:&[i64],queries:&[(usize,usize)])->Vec<usize>{
    let b=((a.len()as f64).sqrt()as usize).max(1);
    let mut order:Vec<usize>=(0..queries.len()).collect();
    order.sort_by(|&i,&j|{
        let(x,y)=(queries[i],queries[j]);(x.0/b).cmp(&(y.0/b)).then_with(||if x.0/b%2==0{
            x.1.cmp(&y.1)
        }
        else{
            y.1.cmp(&x.1)
        }         )
    }     );
    let(mut f,mut l,mut r,mut ans)=(HashMap::new(),0,0,vec![0;queries.len()]);
    for id in order{
        let(ql,qr)=(queries[id].0,queries[id].1+1);
        while l>ql{
            l-=1;
            mo_adjust(&mut f,a[l],1);
        }
        while r<qr{
            mo_adjust(&mut f,a[r],1);
            r+=1;
        }
        while l<ql{
            mo_adjust(&mut f,a[l],-1);
            l+=1;
        }
        while r>qr{
            r-=1;
            mo_adjust(&mut f,a[r],-1);
        }
        ans[id]=f.len();
    }
    ans
}
```

## 带修莫队：区间不同元素数

带单点赋值的区间不同元素数，零基闭区间。Go 操作 Kind=0 查询 [X,Y]、Kind=1 把 X 赋为 Y；TS 使用 [kind,x,y]；Rust 使用 Operation 枚举。按查询出现顺序返回，时间回滚交换旧值和新值。典型 O(n^(5/3))，依赖上一节频率更新。

```rust
enum Operation {
    Query(usize,usize),Update(usize,i64)
}
fn mo_change(a:&mut [i64],change:&mut(usize,i64),l:usize,r:usize,f:&mut HashMap<i64,usize>){
    let p=change.0;
    if l<=p&&p<r{
        mo_adjust(f,a[p],-1);
        mo_adjust(f,change.1,1);
    }
    std::mem::swap(&mut a[p],&mut change.1);
}
fn mo_distinct_with_updates(initial:&[i64],ops:&[Operation])->Vec<usize>{
    let mut a=initial.to_vec();
    let b=((a.len().max(1)as f64).powf(2.0/3.0)as usize).max(1);
    let(mut changes,mut queries)=(vec![],vec![]);
    for op in ops{
        match *op{
            Operation::Query(l,r)=>queries.push((l,r+1,changes.len(),queries.len())),Operation::Update(p,v)=>changes.push((p,v))
        }
    }
    queries.sort_unstable_by_key(|q|(q.0/b,q.1/b,q.2));
    let(mut f,mut l,mut r,mut t,mut ans)=(HashMap::new(),0,0,0,vec![0;queries.len()]);
    for(ql,qr,qt,id)in queries{
        while t<qt{
            mo_change(&mut a,&mut changes[t],l,r,&mut f);
            t+=1;
        }
        while t>qt{
            t-=1;
            mo_change(&mut a,&mut changes[t],l,r,&mut f);
        }
        while l>ql{
            l-=1;
            mo_adjust(&mut f,a[l],1);
        }
        while r<qr{
            mo_adjust(&mut f,a[r],1);
            r+=1;
        }
        while l<ql{
            mo_adjust(&mut f,a[l],-1);
            l+=1;
        }
        while r>qr{
            r-=1;
            mo_adjust(&mut f,a[r],-1);
        }
        ans[id]=f.len();
    }
    ans
}
```

