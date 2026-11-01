# TypeScript 算法模板

与 [C++](./CppTem.md)、[Java](./JavaTem.md)、[Python](./PythonTem.md) 按知识点对应的 49 个算法小节。导航：[README](./README.md) · [C++](./CppTem.md) · [Java](./JavaTem.md) · [Python](./PythonTem.md) · [Go](./GoTem.md) · [TypeScript](./TypeScriptTem.md) · [Rust](./RustTem.md)。

## 使用约定

- 默认数组、图节点从 **0** 开始；树状数组从 **1** 开始。区间开闭、适用条件与依赖见各节。只复制所需实现与依赖，再补题目输入输出。
- 输入须满足有效下标与各节前提；浮点高斯消元使用容差。哈希比较可能碰撞。
- 目标 ES2020 及以上，启用严格类型检查。`number` 实现要求输入、所有中间整数与最终结果都在 `Number.MAX_SAFE_INTEGER` 范围内；精确大整数及模运算使用 `bigint`，两种类型不可直接混算。
- 整数下标须小于 2³¹（树状数组使用 32 位位运算）；普通整数算法不使用位运算替代除法。
- 文件中的函数与类可直接放到一个 `.ts` 文件，无第三方运行时依赖。Node.js 读取标准输入可使用 `node:fs`；对应类型检查需要 Node 类型声明，本文算法本身不依赖它。

## 内容导航

[基础算法](#基础算法) · [数据结构](#数据结构) · [搜索与图论](#搜索与图论) · [数学知识](#数学知识) · [贪心](#贪心) · [动态规划](#动态规划) · [高精度](#高精度) · [莫队](#莫队)

# 基础算法

## 快速输入

批量读取整数。Go 返回读取或解析错误；TypeScript 的解析函数接收输入字符串；Rust 读取标准输入。输入含 ASCII 空白，整数须在对应类型范围内。

```typescript
function parseInts(input: string): number[] {
    if (!input.trim()) return [];
    return input.trim().split(/\s+/).map(s => {
        const n = Number(s);         if (!/^[+-]?\d+$/.test(s) || !Number.isSafeInteger(n)) throw new Error("unsafe integer");         return n;
    }     );
}
// Node.js 入口可调用 parseInts(readFileSync(0, "utf8"))。
// 超过安全整数范围的输入请直接用 BigInt(s)，不要先转 number。
```

## 排序：快速排序与归并排序

快速排序原地修改，最坏 O(n²)；归并排序 O(n log n)。Rust 切片版空数组安全。

```typescript
function quickSort(a: number[], l = 0, r = a.length - 1): void {
    const stack: [number, number][] = [[l, r]];
    while (stack.length) {
        const [lo, hi] = stack.pop()!;
        if (lo >= hi) continue;
        let i = lo - 1, j = hi + 1;
        const x = a[Math.floor((lo + hi) / 2)];
        while (i < j) {
            do {
                i++;
            }
            while (a[i] < x);
            do {
                j--;
            }
            while (a[j] > x);
            if (i < j) [a[i], a[j]] = [a[j], a[i]];
        }
        stack.push([lo, j], [j + 1, hi]);
    }
}
function mergeSort(a: number[]): void {
    const tmp = new Array<number>(a.length);
    function sort(l: number, r: number): void {
        if (r - l <= 1) return;
        const m = Math.floor((l + r) / 2);
        sort(l, m);
        sort(m, r);
        let i = l, j = m, k = l;
        while (i < m && j < r) tmp[k++] = a[i] <= a[j] ? a[i++] : a[j++];
        while (i < m) tmp[k++] = a[i++];
        while (j < r) tmp[k++] = a[j++];
        for (k = l; k < r; k++) a[k] = tmp[k];
    }
    sort(0, a.length);
}
```

## 前缀和：一维与二维

一维查询 [l,r)，二维查询 [x1,x2)×[y1,y2)。O(n)/O(nm) 构建，O(1) 查询。

```typescript
function prefix(a: number[]): number[] {
    const s = [0];
    for (const x of a) s.push(s[s.length-1]+x);
    return s;
}
function prefix2D(a: number[][]): number[][] {
    const n = a.length, m = a[0]?.length ?? 0, s = Array.from({
        length:n+1
    }     , () => Array(m+1).fill(0));
    for (let i=0;i<n;i++) for(let j=0;j<m;j++) s[i+1][j+1]=a[i][j]+s[i][j+1]+s[i+1][j]-s[i][j];
    return s;
}
function rectSum(s: number[][], x1:number,y1:number,x2:number,y2:number):number {
    return s[x2][y2]-s[x1][y2]-s[x2][y1]+s[x1][y1];
}
```

## 差分：一维与二维

在零数组上区间加，更新左闭右开；差分数组比原数组多一位，二维多一行一列。

```typescript
function rangeAdd(d:number[],l:number,r:number,v:number):void {
    d[l]+=v;
    d[r]-=v;
}
function restore(d:number[]):number[] {
    let s=0;
    return d.slice(0,-1).map(x=>s+=x);
}
function rectAdd(d:number[][],x1:number,y1:number,x2:number,y2:number,v:number):void {
    d[x1][y1]+=v;
    d[x2][y1]-=v;
    d[x1][y2]-=v;
    d[x2][y2]+=v;
}
function restore2D(d:number[][]):number[][] {
    const a=Array.from({
        length:d.length-1
    }     ,()=>Array(d[0].length-1).fill(0));
    for(let i=0;i<a.length;i++) for(let j=0;j<a[i].length;j++) a[i][j]=d[i][j]+(i?a[i-1][j]:0)+(j?a[i][j-1]:0)-(i&&j?a[i-1][j-1]:0);
    return a;
}
```

## 二分：整数边界与浮点数

有序数组第一个 ≥x / >x 的位置，不存在返回 n；浮点二分求有限非负数平方根。

```typescript
function lowerBound(a:number[],x:number):number {
    let l=0,r=a.length;
    while(l<r){
        const m=Math.floor((l+r)/2);
        if(a[m]>=x)r=m;
        else l=m+1;
    }
    return l;
}
function upperBound(a:number[],x:number):number {
    let l=0,r=a.length;
    while(l<r){
        const m=Math.floor((l+r)/2);
        if(a[m]>x)r=m;
        else l=m+1;
    }
    return l;
}
function sqrtBinary(x:number):number {
    if(x<0||!Number.isFinite(x))throw new Error("invalid x");
    let l=0,r=Math.max(1,x);
    for(let i=0;i<1100;i++){
        const m=l+(r-l)/2;
        if(m===0||m<=x/m)l=m;
        else r=m;
    }
    return l;
}
```

## 位运算

lowbit 输入正整数，popcount 统计非负整数二进制 1 的数量。TypeScript 使用 bigint，避免位运算隐式截断为 32 位。

```typescript
function lowbit(x:bigint):bigint{
    return x & -x;
}
function popcount(x:bigint):number{
    if(x<0n)throw new Error("nonnegative required");
    let n=0;
    while(x){
        x &= x-1n;
        n++;
    }
    return n;
}
```

## 区间合并

闭区间合并，端点相接视为重叠。O(n log n)，不修改原数组。

```typescript
function mergeIntervals(ranges:[number,number][]):[number,number][]{
    const a=ranges.map(x=>[...x] as [number,number]).sort((x,y)=>x[0]-y[0]),res:[number,number][]=[];
    for(const x of a){
        if(!res.length||res[res.length-1][1]<x[0])res.push(x);
        else res[res.length-1][1]=Math.max(res[res.length-1][1],x[1]);
    }
    return res;
}
```

## 离散化

返回排序去重后的值与原数组的零基排名，O(n log n)。

```typescript
function compress(a:number[]):{
    values:number[],ranks:number[]
}
{
    const values=[...new Set(a)].sort((x,y)=>x-y),rank=new Map(values.map((x,i)=>[x,i]));
    return {
        values,ranks:a.map(x=>rank.get(x)!)
    }     ;
}
```

# 数据结构

## 数组模拟单链表与双链表

数组模拟链表。0 为单链表头哨兵；双链表用 0、1 为左右哨兵。插入返回稳定编号，删除不复用编号。O(1) 更新，删除须针对有效节点。

```typescript
class SingleList {
    value=[0];
    next=[-1];
    insertAfter(k:number,x:number):number{
        const i=this.value.length;
        this.value.push(x);
        this.next.push(this.next[k]);
        this.next[k]=i;
        return i;
    }
    removeAfter(k:number):void{
        if(this.next[k]!==-1)this.next[k]=this.next[this.next[k]];
    }
}
class DoubleList {
    value=[0,0];
    left=[-1,0];
    right=[1,-1];
    insertAfter(k:number,x:number):number{
        const i=this.value.length;
        this.value.push(x);
        this.left.push(k);
        this.right.push(this.right[k]);
        this.left[this.right[k]]=i;
        this.right[k]=i;
        return i;
    }
    remove(k:number):void{
        this.right[this.left[k]]=this.right[k];
        this.left[this.right[k]]=this.left[k];
    }
}
```

## 栈、队列、堆与标准集合

栈、队列与最小堆。队列避免从动态数组头部逐项删除；最小堆用于后文 Dijkstra、哈夫曼与区间分组。Go 的 PairHeap 使用标准 container/heap；TypeScript 的 Heap 为可传比较器的通用堆。

```typescript
class Heap<T> {
    data:T[]=[];
    compare:(a:T,b:T)=>number;
    constructor(compare:(a:T,b:T)=>number){
        this.compare=compare;
    }
    get size():number{
        return this.data.length;
    }
    peek():T{
        if(!this.size)throw new Error("empty heap");
        return this.data[0];
    }
    push(x:T):void{
        const a=this.data;
        a.push(x);
        let i=a.length-1;
        while(i){
            const p=Math.floor((i-1)/2);
            if(this.compare(a[p],a[i])<=0)break;
            [a[p],a[i]]=[a[i],a[p]];
            i=p;
        }
    }
    pop():T{
        if(!this.size)throw new Error("empty heap");
        const a=this.data,ans=a[0],last=a.pop()!;
        if(!a.length)return ans;
        a[0]=last;
        let i=0;
        while(2*i+1<a.length){
            let j=2*i+1;
            if(j+1<a.length&&this.compare(a[j+1],a[j])<0)j++;
            if(this.compare(a[i],a[j])<=0)break;
            [a[i],a[j]]=[a[j],a[i]];
            i=j;
        }
        return ans;
    }
}
function collectionsExample():void{
    const stack=[1];
    stack.push(2);
    stack.pop();
    const queue=[1,2];
    let head=0;
    head++;
    // 以索引出队
    const h=new Heap<number>((a,b)=>a-b);
    h.push(3);
    h.pop();
    const freq=new Map<number,number>();
    freq.set(3,(freq.get(3)??0)+1);
    const seen=new Set([3]);
}
```

## 单调栈与单调队列

左侧最近严格较小下标（不存在 -1）与窗口最小值，O(n)。窗口须满足 1≤k≤n。

```typescript
function previousLess(a:number[]):number[]{
    const st:number[]=[],res:number[]=[];
    for(let i=0;i<a.length;i++){
        while(st.length&&a[st[st.length-1]]>=a[i])st.pop();
        res.push(st.length?st[st.length-1]:-1);
        st.push(i);
    }
    return res;
}
function windowMin(a:number[],k:number):number[]{
    if(k<1||k>a.length)throw new Error("invalid window");
    const q:number[]=[],res:number[]=[];
    let head=0;
    for(let i=0;i<a.length;i++){
        while(head<q.length&&q[head]<=i-k)head++;
        while(head<q.length&&a[q[q.length-1]]>=a[i])q.pop();
        q.push(i);
        if(i>=k-1)res.push(a[q[head]]);
    }
    return res;
}
```

## KMP 字符串匹配

返回全部匹配起点，支持重叠，空模式匹配每个边界。O(n+m)。Go/Rust 按 UTF-8 字节，TypeScript 按 UTF-16 码元编号。

```typescript
function kmp(text:string,pattern:string):number[]{
    const m=pattern.length,res:number[]=[];
    if(!m)return Array.from({
        length:text.length+1
    }     ,(_,i)=>i);
    const pi=Array(m).fill(0);
    for(let i=1,j=0;i<m;i++){
        while(j&&pattern[i]!==pattern[j])j=pi[j-1];
        if(pattern[i]===pattern[j])j++;
        pi[i]=j;
    }
    for(let i=0,j=0;i<text.length;i++){
        while(j&&text[i]!==pattern[j])j=pi[j-1];
        if(text[i]===pattern[j])j++;
        if(j===m){
            res.push(i-m+1);
            j=pi[j-1];
        }
    }
    return res;
}
```

## Trie 字典树

插入单词并统计完整单词出现次数，O(单词长度)。这里限定小写英文字母。

```typescript
class Trie {
    nodes:{
        next:Map<string,number>,count:number
    }     []=[{
        next:new Map(),count:0
    }     ];
    insert(s:string):void{
        let p=0;
        for(const c of s){
            if(!this.nodes[p].next.has(c)){
                this.nodes[p].next.set(c,this.nodes.length);
                this.nodes.push({
                    next:new Map(),count:0
                }                 );
            }
            p=this.nodes[p].next.get(c)!;
        }
        this.nodes[p].count++;
    }
    count(s:string):number{
        let p=0;
        for(const c of s){
            const next=this.nodes[p].next.get(c);
            if(next===undefined)return 0;
            p=next;
        }
        return this.nodes[p].count;
    }
}
```

## 并查集

路径压缩与按集合大小合并，均摊 O(α(n))。重复合并不改变大小。

```typescript
class DSU {
    parent:number[];
    size:number[];
    constructor(n:number){
        this.parent=Array.from({
            length:n
        }         ,(_,i)=>i);
        this.size=Array(n).fill(1);
    }
    find(x:number):number{
        while(this.parent[x]!==x){
            this.parent[x]=this.parent[this.parent[x]];
            x=this.parent[x];
        }
        return x;
    }
    union(a:number,b:number):boolean{
        a=this.find(a);
        b=this.find(b);
        if(a===b)return false;
        if(this.size[a]<this.size[b])[a,b]=[b,a];
        this.parent[b]=a;
        this.size[a]+=this.size[b];
        return true;
    }
}
```

## 可修改、可删除的编号堆

编号唯一的最小堆，支持插入、按编号修改和删除，O(log n)。peek 返回 {值,编号}；操作须针对存在的编号，peek 前须非空。

```typescript
class IndexedHeap {
    data:[number,number][]=[];
    pos=new Map<number,number>();
    swap(i:number,j:number):void{
        [this.data[i],this.data[j]]=[this.data[j],this.data[i]];
        this.pos.set(this.data[i][1],i);
        this.pos.set(this.data[j][1],j);
    }
    up(i:number):number{
        while(i){
            const p=Math.floor((i-1)/2);
            if(this.data[p][0]<=this.data[i][0])break;
            this.swap(i,p);
            i=p;
        }
        return i;
    }
    down(i:number):void{
        while(2*i+1<this.data.length){
            let j=2*i+1;
            if(j+1<this.data.length&&this.data[j+1][0]<this.data[j][0])j++;
            if(this.data[i][0]<=this.data[j][0])break;
            this.swap(i,j);
            i=j;
        }
    }
    insert(id:number,value:number):void{
        if(this.pos.has(id))throw new Error("duplicate id");
        this.pos.set(id,this.data.length);
        this.data.push([value,id]);
        this.up(this.data.length-1);
    }
    change(id:number,value:number):void{
        const i=this.pos.get(id)!;
        this.data[i][0]=value;
        this.down(this.up(i));
    }
    remove(id:number):void{
        const i=this.pos.get(id)!;
        this.swap(i,this.data.length-1);
        this.data.pop();
        this.pos.delete(id);
        if(i<this.data.length)this.down(this.up(i));
    }
    peek():[number,number]{
        return [...this.data[0]];
    }
}
```

## 哈希：开放寻址与链表法

开放寻址与链表法整数集合，仅插入和查询。容量为正，开放寻址容量须大于不同元素数。独立占用标记支持任意键值。

```typescript
class OpenHash {
    keys:(number|undefined)[];
    constructor(n:number){
        this.keys=Array(n).fill(undefined);
    }
    locate(x:number):number{
        const n=this.keys.length;
        let k=((x%n)+n)%n;
        const start=k;
        while(this.keys[k]!==undefined&&this.keys[k]!==x){
            k=(k+1)%n;
            if(k===start)throw new Error("full table");
        }
        return k;
    }
    add(x:number):void{
        this.keys[this.locate(x)]=x;
    }
    contains(x:number):boolean{
        return this.keys[this.locate(x)]!==undefined;
    }
}
class ChainHash {
    buckets:number[][];
    constructor(n:number){
        this.buckets=Array.from({
            length:n
        }         ,()=>[]);
    }
    bucket(x:number):number{
        const n=this.buckets.length;
        return ((x%n)+n)%n;
    }
    contains(x:number):boolean{
        return this.buckets[this.bucket(x)].includes(x);
    }
    add(x:number):void{
        if(!this.contains(x))this.buckets[this.bucket(x)].push(x);
    }
}
```

## 字符串前缀哈希

前缀哈希查询 [l,r)，O(n) 构建、O(1) 查询。base=131，mod=1000000007，有碰撞可能；精确比较需再核对原字符串。Go/Rust 按字节，TypeScript 按 UTF-16 码元。

```typescript
class StringHash {
    h=[0n];
    power=[1n];
    mod=1000000007n;
    constructor(s:string){
        for(let i=0;i<s.length;i++){
            this.h.push((this.h[i]*131n+BigInt(s.charCodeAt(i))+1n)%this.mod);
            this.power.push(this.power[i]*131n%this.mod);
        }
    }
    get(l:number,r:number):bigint{
        return (this.h[r]-this.h[l]*this.power[r-l]%this.mod+this.mod)%this.mod;
    }
}
```

## 二叉树遍历

一般二叉树的前、中、后序遍历，O(n)。Go/TypeScript 使用节点引用；Rust 使用节点数组与 Option<usize> 子节点索引。树须无环，迭代遍历。

```typescript
class TreeNode {
    value:number;
    left:TreeNode|null=null;
    right:TreeNode|null=null;
    constructor(value:number){
        this.value=value;
    }
}
function traversals(root:TreeNode|null):[number[],number[],number[]]{
    const pre:number[]=[],inorder:number[]=[],post:number[]=[],st:TreeNode[]=[];
    if(root)st.push(root);
    while(st.length){
        const u=st.pop()!;
        pre.push(u.value);
        if(u.right)st.push(u.right);
        if(u.left)st.push(u.left);
    }
    let cur=root;
    while(cur||st.length){
        while(cur){
            st.push(cur);
            cur=cur.left;
        }
        cur=st.pop()!;
        inorder.push(cur.value);
        cur=cur.right;
    }
    if(root)st.push(root);
    while(st.length){
        const u=st.pop()!;
        post.push(u.value);
        if(u.left)st.push(u.left);
        if(u.right)st.push(u.right);
    }
    return [pre,inorder,post.reverse()];
}
```

## 哈夫曼树与合并代价

非负权值的最小合并代价，即哈夫曼带权路径长度。空集合或单节点为 0，O(n log n)。Go/TypeScript 依赖前文 PairHeap / Heap。

```typescript
function huffmanCost(a:number[]):number{
    const h=new Heap<number>((x,y)=>x-y);
    for(const x of a)h.push(x);
    let ans=0;
    while(h.size>1){
        const s=h.pop()+h.pop();
        ans+=s;
        h.push(s);
    }
    return ans;
}
```

## 树状数组：点加、区间加与区间和

本节一基下标。点加、前缀和与区间和；双树状数组支持闭区间 [l,r] 加与区间和。初始为零，O(log n)。

```typescript
class Fenwick {
    tree:number[];
    constructor(n:number){
        this.tree=Array(n+1).fill(0);
    }
    add(i:number,v:number):void{
        if(i<=0)throw new Error("index starts at 1");
        for(;i<this.tree.length;i+=i&-i)this.tree[i]+=v;
    }
    prefix(i:number):number{
        let s=0;
        for(;i>0;i-=i&-i)s+=this.tree[i];
        return s;
    }
    rangeSum(l:number,r:number):number{
        return this.prefix(r)-this.prefix(l-1);
    }
}
class RangeFenwick {
    b1:Fenwick;
    b2:Fenwick;
    constructor(n:number){
        this.b1=new Fenwick(n);
        this.b2=new Fenwick(n);
    }
    rangeAdd(l:number,r:number,v:number):void{
        this.b1.add(l,v);
        this.b1.add(r+1,-v);
        this.b2.add(l,v*(l-1));
        this.b2.add(r+1,-v*r);
    }
    prefix(i:number):number{
        return this.b1.prefix(i)*i-this.b2.prefix(i);
    }
    rangeSum(l:number,r:number):number{
        return this.prefix(r)-this.prefix(l-1);
    }
}
```

## 线段树：懒标记区间加、区间和

非空数组，零基闭区间 [l,r]。懒标记支持区间加与区间和，O(log n)。

```typescript
class SegmentTree {
    n:number;
    sum:number[];
    lazy:number[];
    constructor(a:number[]){
        this.n=a.length;
        if(!this.n)throw new Error("empty array");
        this.sum=Array(4*this.n).fill(0);
        this.lazy=Array(4*this.n).fill(0);
        this.build(1,0,this.n-1,a);
    }
    build(u:number,l:number,r:number,a:number[]):void{
        if(l===r){
            this.sum[u]=a[l];
            return;
        }
        const m=Math.floor((l+r)/2);
        this.build(2*u,l,m,a);
        this.build(2*u+1,m+1,r,a);
        this.sum[u]=this.sum[2*u]+this.sum[2*u+1];
    }
    apply(u:number,l:number,r:number,v:number):void{
        this.sum[u]+=(r-l+1)*v;
        this.lazy[u]+=v;
    }
    push(u:number,l:number,r:number):void{
        const m=Math.floor((l+r)/2);
        this.apply(2*u,l,m,this.lazy[u]);
        this.apply(2*u+1,m+1,r,this.lazy[u]);
        this.lazy[u]=0;
    }
    add(ql:number,qr:number,v:number,u=1,l=0,r=this.n-1):void{
        if(ql<=l&&r<=qr){
            this.apply(u,l,r,v);
            return;
        }
        this.push(u,l,r);
        const m=Math.floor((l+r)/2);
        if(ql<=m)this.add(ql,qr,v,2*u,l,m);
        if(qr>m)this.add(ql,qr,v,2*u+1,m+1,r);
        this.sum[u]=this.sum[2*u]+this.sum[2*u+1];
    }
    query(ql:number,qr:number,u=1,l=0,r=this.n-1):number{
        if(ql<=l&&r<=qr)return this.sum[u];
        this.push(u,l,r);
        const m=Math.floor((l+r)/2);
        let ans=0;
        if(ql<=m)ans+=this.query(ql,qr,2*u,l,m);
        if(qr>m)ans+=this.query(ql,qr,2*u+1,m+1,r);
        return ans;
    }
}
```

## 树形 DP：最大权独立集

非空无向树，最大权独立集：相邻节点不能同时选，可全不选。迭代建树后逆序转移，O(n)。

```typescript
function treeIndependentSet(g:number[][],w:number[]):number{
    const p=Array(g.length).fill(-2),order=[0],f=Array.from({
        length:g.length
    }     ,()=>[0,0]);
    p[0]=-1;
    for(let i=0;i<order.length;i++)for(const v of g[order[i]])if(p[v]===-2){
        p[v]=order[i];
        order.push(v);
    }
    for(let i=order.length-1;i>=0;i--){
        const u=order[i];
        f[u][1]+=w[u];
        if(p[u]>=0){
            f[p[u]][0]+=Math.max(...f[u]);
            f[p[u]][1]+=f[u][0];
        }
    }
    return Math.max(...f[0]);
}
```

# 搜索与图论

## DFS：全排列与 N 皇后

返回全排列与 N 皇后的全部解；内存随解数增长，仅用于搜索深度较小的题。皇后解记录各行列号。

```typescript
function permutations(n:number):number[][]{
    const ans:number[][]=[],path:number[]=[],used=Array(n).fill(false);
    function dfs():void{
        if(path.length===n){
            ans.push([...path]);
            return;
        }
        for(let x=0;x<n;x++)if(!used[x]){
            used[x]=true;
            path.push(x);
            dfs();
            path.pop();
            used[x]=false;
        }
    }
    dfs();
    return ans;
}
function nQueens(n:number):number[][]{
    const ans:number[][]=[],path:number[]=[],col=new Set<number>(),d1=new Set<number>(),d2=new Set<number>();
    function dfs(r:number):void{
        if(r===n){
            ans.push([...path]);
            return;
        }
        for(let c=0;c<n;c++)if(!col.has(c)&&!d1.has(r+c)&&!d2.has(r-c)){
            col.add(c);
            d1.add(r+c);
            d2.add(r-c);
            path.push(c);
            dfs(r+1);
            path.pop();
            col.delete(c);
            d1.delete(r+c);
            d2.delete(r-c);
        }
    }
    dfs(0);
    return ans;
}
```

## 树的重心

树的重心指标：删除单个节点后最大连通块大小的最小值；非空无向树，O(n)。

```typescript
function centroidBalance(g:number[][]):number{
    const n=g.length,p=Array(n).fill(-2),size=Array(n).fill(1),order=[0];
    p[0]=-1;
    for(let i=0;i<order.length;i++)for(const v of g[order[i]])if(p[v]===-2){
        p[v]=order[i];
        order.push(v);
    }
    let best=n;
    for(let i=n-1;i>=0;i--){
        const u=order[i];
        let largest=0;
        for(const v of g[u])if(p[v]===u){
            size[u]+=size[v];
            largest=Math.max(largest,size[v]);
        }
        best=Math.min(best,Math.max(largest,n-size[u]));
    }
    return best;
}
```

## BFS：图的层次与迷宫

无权图距离，不可达 -1；迷宫 0 可走、1 障碍，起终点为左上与右下。O(V+E) / O(nm)。

```typescript
function bfs(g:number[][],source:number):number[]{
    const d=Array(g.length).fill(-1),q=[source];
    d[source]=0;
    for(let h=0;h<q.length;h++){
        const u=q[h];
        for(const v of g[u])if(d[v]===-1){
            d[v]=d[u]+1;
            q.push(v);
        }
    }
    return d;
}
function mazeDistance(a:number[][]):number{
    if(!a.length||!a[0].length||a[0][0]||a[a.length-1][a[0].length-1])return -1;
    const n=a.length,m=a[0].length,d=Array.from({
        length:n
    }     ,()=>Array(m).fill(-1)),q:[[number,number]]|[number,number][]=[[0,0]];
    d[0][0]=0;
    for(let h=0;h<q.length;h++){
        const [x,y]=q[h];
        for(const [dx,dy]of [[-1,0],[0,1],[1,0],[0,-1]]){
            const nx=x+dx,ny=y+dy;
            if(nx>=0&&nx<n&&ny>=0&&ny<m&&a[nx][ny]===0&&d[nx][ny]===-1){
                d[nx][ny]=d[x][y]+1;
                q.push([nx,ny]);
            }
        }
    }
    return d[n-1][m-1];
}
```

## Dijkstra：朴素与堆优化

边权非负。矩阵无边用 INF（Go/Rust 的最大 i64 /4）或 Infinity；邻接项为 {终点,权重}。朴素 O(n²)，堆优化 O((V+E) log V)。Go/TypeScript 依赖 PairHeap / Heap。

```typescript
type Edge = [number,number];
// [终点,权重]
function dijkstraDense(g:number[][],s:number):number[]{
    const n=g.length,d=Array(n).fill(Infinity),used=Array(n).fill(false);
    d[s]=0;
    for(let i=0;i<n;i++){
        let u=-1;
        for(let j=0;j<n;j++)if(!used[j]&&(u===-1||d[j]<d[u]))u=j;
        if(u===-1||d[u]===Infinity)break;
        used[u]=true;
        for(let v=0;v<n;v++)d[v]=Math.min(d[v],d[u]+g[u][v]);
    }
    return d;
}
function dijkstra(g:Edge[][],s:number):number[]{
    const d=Array(g.length).fill(Infinity),h=new Heap<[number,number]>((a,b)=>a[0]-b[0]);
    d[s]=0;
    h.push([0,s]);
    while(h.size){
        const [dist,u]=h.pop();
        if(dist!==d[u])continue;
        for(const [v,w]of g[u])if(d[v]>dist+w){
            d[v]=dist+w;
            h.push([d[v],v]);
        }
    }
    return d;
}
```

## Bellman-Ford：最多 k 条边

边表 {u,v,w}，允许负权；最多 k 条边，每轮使用上一轮副本。不可达 INF / Infinity，O(kE)。Go/Rust 依赖前文 INF。

```typescript
type Arc=[number,number,number];
function bellmanFord(n:number,edges:Arc[],s:number,k:number):number[]{
    const d=Array(n).fill(Infinity);
    d[s]=0;
    for(let i=0;i<k;i++){
        const old=[...d];
        for(const [u,v,w]of edges)if(old[u]!==Infinity)d[v]=Math.min(d[v],old[u]+w);
    }
    return d;
}
```

## SPFA

允许负权；源点可达负环时 Go 返回 false、TypeScript 抛异常、Rust 返回 None。不可达 INF / Infinity，最坏 O(VE)。依赖 Edge 与 INF（TypeScript 不需要 INF）。

```typescript
function spfa(g:Edge[][],s:number):number[]{
    const n=g.length,d=Array(n).fill(Infinity),count=Array(n).fill(0),queued=Array(n).fill(false),q=[s];
    d[s]=0;
    queued[s]=true;
    for(let h=0;h<q.length;h++){
        const u=q[h];
        queued[u]=false;
        for(const [v,w]of g[u])if(d[v]>d[u]+w){
            d[v]=d[u]+w;
            count[v]=count[u]+1;
            if(count[v]>=n)throw new Error("reachable negative cycle");
            if(!queued[v]){
                queued[v]=true;
                q.push(v);
            }
        }
    }
    return d;
}
```

## Floyd 全源最短路

无负环；对角线 0，无边 INF / Infinity，重边建图取最小值。返回副本，O(n³)。Go/Rust 依赖 INF。

```typescript
function floyd(g:number[][]):number[][]{
    const d=g.map(row=>[...row]);
    for(let k=0;k<d.length;k++)for(let i=0;i<d.length;i++)if(d[i][k]!==Infinity)for(let j=0;j<d.length;j++)if(d[k][j]!==Infinity)d[i][j]=Math.min(d[i][j],d[i][k]+d[k][j]);
    return d;
}
```

## 最小生成树：Prim 与 Kruskal

无向图，允许负权。Prim 使用矩阵（无边 INF / Infinity），O(n²)；Kruskal 边表使用 Arc，O(E log E)，依赖 DSU。非连通为 Go false、TS null、Rust None，空图代价 0。

```typescript
function prim(g:number[][]):number|null{
    const n=g.length;
    if(!n)return 0;
    const d=Array(n).fill(Infinity),used=Array(n).fill(false);
    d[0]=0;
    let ans=0;
    for(let i=0;i<n;i++){
        let u=-1;
        for(let j=0;j<n;j++)if(!used[j]&&(u===-1||d[j]<d[u]))u=j;
        if(d[u]===Infinity)return null;
        ans+=d[u];
        used[u]=true;
        for(let v=0;v<n;v++)if(!used[v])d[v]=Math.min(d[v],g[u][v]);
    }
    return ans;
}
function kruskal(n:number,edges:Arc[]):number|null{
    const d=new DSU(n);
    let ans=0,count=0;
    for(const [u,v,w]of [...edges].sort((a,b)=>a[2]-b[2]))if(d.union(u,v)){
        ans+=w;
        count++;
    }
    return n===0||count===n-1?ans:null;
}
```

## 二分图：染色与匈牙利算法

无向图染色 O(V+E)。最大匹配输入仅左点邻接表，邻接值为右点编号；迭代增广 O(VE)，不受递归栈深度限制。

```typescript
function isBipartite(g:number[][]):boolean{
    const color=Array(g.length).fill(0);
    for(let s=0;s<g.length;s++){
        if(color[s])continue;
        color[s]=1;
        const q=[s];
        for(let h=0;h<q.length;h++){
            const u=q[h];
            for(const v of g[u]){
                if(color[v]===color[u])return false;
                if(!color[v]){
                    color[v]=3-color[u];
                    q.push(v);
                }
            }
        }
    }
    return true;
}
function maxMatching(g:number[][],nr:number):number{
    const left=Array(g.length).fill(-1),right=Array(nr).fill(-1);
    let ans=0;
    for(let s=0;s<g.length;s++){
        const prev=Array(nr).fill(-1),seen=Array(g.length).fill(false),q=[s];
        seen[s]=true;
        let end=-1;
        for(let h=0;h<q.length&&end===-1;h++){
            const u=q[h];
            for(const v of g[u]){
                if(prev[v]!==-1)continue;
                prev[v]=u;
                if(right[v]===-1){
                    end=v;
                    break;
                }
                if(!seen[right[v]]){
                    seen[right[v]]=true;
                    q.push(right[v]);
                }
            }
        }
        if(end!==-1){
            ans++;
            while(end!==-1){
                const u=prev[end],old=left[u];
                left[u]=end;
                right[end]=u;
                end=old;
            }
        }
    }
    return ans;
}
```

## 拓扑排序

有向图拓扑排序，O(V+E)。存在环时 Go 返回 false、TypeScript null、Rust None。

```typescript
function topologicalSort(g:number[][]):number[]|null{
    const deg=Array(g.length).fill(0),q:number[]=[];
    for(const row of g)for(const v of row)deg[v]++;
    for(let i=0;i<g.length;i++)if(!deg[i])q.push(i);
    for(let h=0;h<q.length;h++)for(const v of g[q[h]])if(--deg[v]===0)q.push(v);
    return q.length===g.length?q:null;
}
```

# 数学知识

## 质数：试除、分解与筛法

试除 O(√n)，分解返回质因子指数表（n≥1）。两种筛均包含上界 n：埃氏 O(n log log n)、线性 O(n)。

```typescript
function isPrime(n:number):boolean{
    if(n<2)return false;
    for(let p=2;p<=Math.floor(n/p);p++)if(n%p===0)return false;
    return true;
}
function factorize(n:number):Map<number,number>{
    if(n<1)throw new Error("positive required");
    const f=new Map<number,number>();
    for(let p=2;p<=Math.floor(n/p);p++)while(n%p===0){
        f.set(p,(f.get(p)??0)+1);
        n/=p;
    }
    if(n>1)f.set(n,(f.get(n)??0)+1);
    return f;
}
function sieve(n:number):number[]{
    const comp=Array(n+1).fill(false),primes:number[]=[];
    for(let i=2;i<=n;i++)if(!comp[i]){
        primes.push(i);
        for(let j=i*i;j<=n;j+=i)comp[j]=true;
    }
    return primes;
}
function linearSieve(n:number):number[]{
    const comp=Array(n+1).fill(false),primes:number[]=[];
    for(let i=2;i<=n;i++){
        if(!comp[i])primes.push(i);
        for(const p of primes){
            if(p>Math.floor(n/i))break;
            comp[i*p]=true;
            if(i%p===0)break;
        }
    }
    return primes;
}
```

## 约数、约数个数与约数之和

枚举正整数约数并排序；约数个数与和从质因子指数表计算。模数须正；Go 限定 mod≤2147483647，Rust 模数用 u32 并用 u128 中间乘法，TypeScript 用 bigint。

```typescript
function divisors(n:number):number[]{
    if(n<1)throw new Error("positive required");
    const a:number[]=[];
    for(let i=1;i<=Math.floor(n/i);i++)if(n%i===0){
        a.push(i);
        if(i!==n/i)a.push(n/i);
    }
    return a.sort((x,y)=>x-y);
}
function divisorCount(f:Map<number,number>,mod:bigint):bigint{
    let ans=1n%mod;
    for(const e of f.values())ans=ans*BigInt(e+1)%mod;
    return ans;
}
function divisorSum(f:Map<number,number>,mod:bigint):bigint{
    let ans=1n%mod;
    for(const [p,e]of f){
        let term=1n%mod;
        for(let i=0;i<e;i++)term=(term*BigInt(p)+1n)%mod;
        ans=ans*term%mod;
    }
    return ans;
}
```

## 欧几里得、扩展欧几里得与线性同余方程

a、b 非负且不同时为 0；扩展欧几里得返回 (g,x,y)，ax+by=g。线性同余方程 ax≡b(mod m)，m>0；无解为 false / null / None。Go/Rust 系数中间值须在 i64 范围内，TS 使用 bigint。

```typescript
function gcd(a:bigint,b:bigint):bigint{
    while(b)[a,b]=[b,a%b];
    return a;
}
function exgcd(a:bigint,b:bigint):[bigint,bigint,bigint]{
    if(!b)return [a,1n,0n];
    const [g,x,y]=exgcd(b,a%b);
    return [g,y,x-a/b*y];
}
function solveCongruence(a:bigint,b:bigint,m:bigint):bigint|null{
    const [g,x]=exgcd(a,m);
    if(b%g)return null;
    const period=m/g;
    return ((x*(b/g))%period+period)%period;
}
```

## 快速幂

指数非负，模数正，O(log b)。Go 模数≤2147483647；Rust 模数 u32，乘法用 u64；TypeScript 使用 bigint。

```typescript
function modPow(a:bigint,b:bigint,mod:bigint):bigint{
    if(b<0n||mod<=0n)throw new Error("invalid arguments");
    a=(a%mod+mod)%mod;
    let res=1n%mod;
    while(b){
        if(b&1n)res=res*a%mod;
        a=a*a%mod;
        b>>=1n;
    }
    return res;
}
```

## 高斯消元

n×(n+1) 增广矩阵，直接修改；0=唯一解、1=无穷多解、2=无解。唯一解存最后一列，O(n³)，eps=1e-9。

```typescript
function gauss(a:number[][]):number{
    const n=a.length,eps=1e-9;
    let r=0;
    for(let c=0;c<n;c++){
        let t=r;
        for(let i=r;i<n;i++)if(Math.abs(a[i][c])>Math.abs(a[t][c]))t=i;
        if(Math.abs(a[t][c])<eps)continue;
        [a[t],a[r]]=[a[r],a[t]];
        for(let j=n;j>=c;j--)a[r][j]/=a[r][c];
        for(let i=r+1;i<n;i++)if(Math.abs(a[i][c])>eps)for(let j=n;j>=c;j--)a[i][j]-=a[r][j]*a[i][c];
        r++;
    }
    if(r<n){
        for(let i=r;i<n;i++)if(Math.abs(a[i][n])>eps)return 2;
        return 1;
    }
    for(let i=n-1;i>=0;i--)for(let j=i+1;j<n;j++)a[i][n]-=a[i][j]*a[j][n];
    return 0;
}
```

## 组合数：递推、阶乘逆元、Lucas 与精确值

递推适用任意正模数。阶乘逆元要求 p 为质数、n<p；Lucas 要求 p 为质数且适合较小 p，小组合 O(p)。依赖 ModPow / modPow / mod_pow。Go 模数≤2147483647；Rust 用 u32；精确组合数 Go 用 math/big、TS 用 bigint，Rust 返回十进制字符串并依赖后文数位运算。

```typescript
function combinationTable(n:number,mod:bigint):bigint[][]{
    const c=Array.from({
        length:n+1
    }     ,()=>Array<bigint>(n+1).fill(0n));
    for(let i=0;i<=n;i++){
        c[i][0]=1n%mod;
        for(let j=1;j<=i;j++)c[i][j]=(c[i-1][j-1]+c[i-1][j])%mod;
    }
    return c;
}
class CombPrime {
    fact:bigint[];
    inv:bigint[];
    p:bigint;
    constructor(n:number,p:bigint){
        if(p<2n||BigInt(n)>=p)throw new Error("require n < prime p");
        this.p=p;
        this.fact=Array(n+1).fill(1n);
        this.inv=Array(n+1).fill(1n);
        for(let i=1;i<=n;i++)this.fact[i]=this.fact[i-1]*BigInt(i)%p;
        this.inv[n]=modPow(this.fact[n],p-2n,p);
        for(let i=n;i>0;i--)this.inv[i-1]=this.inv[i]*BigInt(i)%p;
    }
    choose(n:number,k:number):bigint{
        return k<0||k>n?0n:this.fact[n]*this.inv[k]%this.p*this.inv[n-k]%this.p;
    }
}
function smallComb(n:bigint,k:bigint,p:bigint):bigint{
    if(k>n)return 0n;
    k=k<n-k?k:n-k;
    let a=1n,b=1n;
    for(let i=1n;i<=k;i++){
        a=a*(n-i+1n)%p;
        b=b*i%p;
    }
    return a*modPow(b,p-2n,p)%p;
}
function lucas(n:bigint,k:bigint,p:bigint):bigint{
    if(p<2n)throw new Error("prime p required");
    let ans=1n;
    while(n||k){
        ans=ans*smallComb(n%p,k%p,p)%p;
        n/=p;
        k/=p;
    }
    return ans;
}
function exactComb(n:number,k:number):bigint{
    if(k<0||k>n)return 0n;
    k=Math.min(k,n-k);
    let ans=1n;
    for(let i=1;i<=k;i++)ans=ans*BigInt(n-i+1)/BigInt(i);
    return ans;
}
```

## 卡特兰数

返回精确卡特兰数 C(2n,n)/(n+1)，n≥0。依赖精确组合数；Rust 还依赖后文数位运算，n 须使 2n 可用 u32 表示。

```typescript
function catalan(n:number):bigint{
    return exactComb(2*n,n)/BigInt(n+1);
}
```

# 贪心

## 合并果子、区间覆盖、分组与选点

合并果子调用前文哈夫曼代价。闭区间覆盖要求 start<end，失败返回 -1；区间分组端点相接冲突；选点可用公共端点覆盖多段。O(n log n)，Go/TS 分组依赖最小堆。

```typescript
function coverIntervals(start:number,end:number,ranges:[number,number][]):number{
    const a=[...ranges].sort((x,y)=>x[0]-y[0]);
    let i=0,count=0;
    while(start<end){
        let next=start;
        while(i<a.length&&a[i][0]<=start){
            next=Math.max(next,a[i][1]);
            i++;
        }
        if(next===start)return -1;
        start=next;
        count++;
    }
    return count;
}
function groupIntervals(ranges:[number,number][]):number{
    const h=new Heap<number>((x,y)=>x-y);
    for(const [l,r]of [...ranges].sort((a,b)=>a[0]-b[0])){
        if(h.size&&h.peek()<l)h.pop();
        h.push(r);
    }
    return h.size;
}
function selectPoints(ranges:[number,number][]):number{
    let point=-Infinity,count=0;
    for(const [l,r]of [...ranges].sort((a,b)=>a[1]-b[1]))if(point<l){
        point=r;
        count++;
    }
    return count;
}
```

# 动态规划

## 背包：01、完全、多重与分组

容量至多 m，可不选；体积为正、数量非负，价值可负。01/完全 O(nm)，多重二进制拆分 O(mΣlog(s+1))，分组 O(mΣ组大小)。

```typescript
type Item=[number,number];
type BoundedItem=[number,number,number];
function knapsack01(m:number,items:Item[]):number{
    const f=Array(m+1).fill(0);
    for(const [v,w]of items)for(let j=m;j>=v;j--)f[j]=Math.max(f[j],f[j-v]+w);
    return f[m];
}
function knapsackComplete(m:number,items:Item[]):number{
    const f=Array(m+1).fill(0);
    for(const [v,w]of items)for(let j=v;j<=m;j++)f[j]=Math.max(f[j],f[j-v]+w);
    return f[m];
}
function knapsackBounded(m:number,items:BoundedItem[]):number{
    const f=Array(m+1).fill(0);
    for(const [v,w,s]of items){
        let remaining=Math.min(s,Math.floor(m/v)),k=1;
        while(remaining){
            const take=Math.min(k,remaining);
            remaining-=take;
            for(let j=m;j>=v*take;j--)f[j]=Math.max(f[j],f[j-v*take]+w*take);
            k*=2;
        }
    }
    return f[m];
}
function knapsackGrouped(m:number,groups:Item[][]):number{
    const f=Array(m+1).fill(0);
    for(const group of groups){
        const old=[...f];
        for(let j=0;j<=m;j++)for(const [v,w]of group)if(j>=v)f[j]=Math.max(f[j],old[j-v]+w);
    }
    return f[m];
}
```

## 线性 DP：数字三角形

三角形第 i 行 i+1 项，最大路径和，可负数；自底向上 O(n²) 时间、O(n) 额外空间。空输入返回 0。

```typescript
function triangleMax(a:number[][]):number{
    if(!a.length)return 0;
    const f=[...a[a.length-1]];
    for(let i=a.length-2;i>=0;i--)for(let j=0;j<=i;j++)f[j]=a[i][j]+Math.max(f[j],f[j+1]);
    return f[0];
}
```

## 最长上升子序列

严格上升，O(n²) 返回一条正向序列；O(n log n) 版本只返回长度。空输入安全。

```typescript
function lis(a:number[]):number[]{
    if(!a.length)return [];
    const f=Array(a.length).fill(1),p=Array(a.length).fill(-1);
    let best=0;
    for(let i=0;i<a.length;i++){
        for(let j=0;j<i;j++)if(a[j]<a[i]&&f[j]+1>f[i]){
            f[i]=f[j]+1;
            p[i]=j;
        }
        if(f[i]>f[best])best=i;
    }
    const ans:number[]=[];
    while(best!==-1){
        ans.push(a[best]);
        best=p[best];
    }
    return ans.reverse();
}
function lisLength(a:number[]):number{
    const tails:number[]=[];
    for(const x of a){
        const i=lowerBound(tails,x);
        if(i===tails.length)tails.push(x);
        else tails[i]=x;
    }
    return tails.length;
}
```

## 最长公共子序列

最长公共子序列长度，O(nm) 时间、O(m) 空间。Go/Rust 按字节，TypeScript 按 UTF-16 码元比较。

```typescript
function lcsLength(a:string,b:string):number{
    const f=Array(b.length+1).fill(0);
    for(let i=0;i<a.length;i++){
        let diagonal=0;
        for(let j=1;j<=b.length;j++){
            const old=f[j];
            f[j]=a[i]===b[j-1]?diagonal+1:Math.max(f[j],f[j-1]);
            diagonal=old;
        }
    }
    return f[b.length];
}
```

## 区间 DP：石子合并

线性排列的石子，每次合并相邻两堆，求最小总代价。O(n³) 时间、O(n²) 空间。

```typescript
function stoneMerge(a:number[]):number{
    const n=a.length;
    if(!n)return 0;
    const s=prefix(a),f=Array.from({
        length:n
    }     ,()=>Array(n).fill(0));
    for(let len=2;len<=n;len++)for(let l=0;l+len<=n;l++){
        const r=l+len-1;
        f[l][r]=Infinity;
        for(let k=l;k<r;k++)f[l][r]=Math.min(f[l][r],f[l][k]+f[k+1][r]+s[r+1]-s[l]);
    }
    return f[0][n-1];
}
```

# 高精度

## 大整数：加、减、乘、除

十进制非负整数的加、减、乘、除；减法允许负结果，除数必须正。Go 使用 math/big；TS 使用 bigint；Rust 依赖下一节数位运算，不需外部 crate。除法返回商与余数，均为字符串。

```typescript
function bigAdd(a:string,b:string):string{
    return (BigInt(a)+BigInt(b)).toString();
}
function bigSubtract(a:string,b:string):string{
    return (BigInt(a)-BigInt(b)).toString();
}
function bigMultiply(a:string,b:string):string{
    return (BigInt(a)*BigInt(b)).toString();
}
function bigDivide(a:string,b:string):[string,string]{
    const d=BigInt(b);
    if(d<=0n)throw new Error("positive divisor required");
    const x=BigInt(a);
    return [(x/d).toString(),(x%d).toString()];
}
```

## 手写高精度：十进制数位数组

数位低位在前，至少一位，无多余高位零。加法、非负减法（A≥B）、乘非负小整数、除正小整数。Go/TS 小整数≤10^9；Go 中间值 int64、TS 中间值仍在安全整数内。Rust 小整数 u64，中间值 u128。

```typescript
function digits(s:string):number[]{
    const a=[...s].reverse().map(Number);
    trimDigits(a);
    return a;
}
function trimDigits(a:number[]):void{
    while(a.length>1&&a[a.length-1]===0)a.pop();
}
function addDigits(a:number[],b:number[]):number[]{
    const c:number[]=[];
    let carry=0;
    for(let i=0;i<Math.max(a.length,b.length)||carry;i++){
        carry+=(a[i]??0)+(b[i]??0);
        c.push(carry%10);
        carry=Math.floor(carry/10);
    }
    return c;
}
function subtractDigits(a:number[],b:number[]):number[]{
    const c:number[]=[];
    let borrow=0;
    for(let i=0;i<a.length;i++){
        let x=a[i]-borrow-(b[i]??0);
        borrow=x<0?1:0;
        if(x<0)x+=10;
        c.push(x);
    }
    trimDigits(c);
    return c;
}
function multiplyDigits(a:number[],b:number):number[]{
    const c:number[]=[];
    let carry=0;
    for(let i=0;i<a.length||carry;i++){
        if(i<a.length)carry+=a[i]*b;
        c.push(carry%10);
        carry=Math.floor(carry/10);
    }
    trimDigits(c);
    return c;
}
function divideDigits(a:number[],b:number):[number[],number]{
    const c=Array(a.length).fill(0);
    let r=0;
    for(let i=a.length-1;i>=0;i--){
        r=r*10+a[i];
        c[i]=Math.floor(r/b);
        r%=b;
    }
    trimDigits(c);
    return [c,r];
}
```

# 莫队

## 普通莫队：区间不同元素数

普通莫队，区间不同元素数；零基闭区间 [l,r]，原查询顺序返回。内部窗口使用 [L,R)；右端点奇偶块排序，典型 O((n+q)√n)。

```typescript
function moAdjust(f:Map<number,number>,x:number,delta:number):void{
    const n=(f.get(x)??0)+delta;
    if(n===0)f.delete(x);
    else f.set(x,n);
}
function moDistinct(a:number[],queries:[number,number][]):number[]{
    const b=Math.max(1,Math.floor(Math.sqrt(a.length))),order=queries.map((_,i)=>i);
    order.sort((i,j)=>{
        const x=queries[i],y=queries[j],c=Math.floor(x[0]/b)-Math.floor(y[0]/b);return c||(Math.floor(x[0]/b)%2===0?x[1]-y[1]:y[1]-x[1]);
    }     );
    const f=new Map<number,number>(),ans=Array(queries.length).fill(0);
    let l=0,r=0;
    for(const id of order){
        const [ql,end]=queries[id],qr=end+1;
        while(l>ql)moAdjust(f,a[--l],1);
        while(r<qr)moAdjust(f,a[r++],1);
        while(l<ql)moAdjust(f,a[l++],-1);
        while(r>qr)moAdjust(f,a[--r],-1);
        ans[id]=f.size;
    }
    return ans;
}
```

## 带修莫队：区间不同元素数

带单点赋值的区间不同元素数，零基闭区间。Go 操作 Kind=0 查询 [X,Y]、Kind=1 把 X 赋为 Y；TS 使用 [kind,x,y]；Rust 使用 Operation 枚举。按查询出现顺序返回，时间回滚交换旧值和新值。典型 O(n^(5/3))，依赖上一节频率更新。

```typescript
function moDistinctWithUpdates(initial:number[],ops:[number,number,number][]):number[]{
    const a=[...initial],b=Math.max(1,Math.floor(Math.max(1,a.length)**(2/3))),changes:[number,number][]=[],queries:[number,number,number,number][]=[];
    for(const [kind,x,y]of ops){
        if(kind===0)queries.push([x,y+1,changes.length,queries.length]);
        else changes.push([x,y]);
    }
    queries.sort((x,y)=>Math.floor(x[0]/b)-Math.floor(y[0]/b)||Math.floor(x[1]/b)-Math.floor(y[1]/b)||x[2]-y[2]);
    const f=new Map<number,number>(),ans=Array(queries.length).fill(0);
    let l=0,r=0,t=0;
    function change(i:number):void{
        const [p,v]=changes[i];
        if(l<=p&&p<r){
            moAdjust(f,a[p],-1);
            moAdjust(f,v,1);
        }         [a[p],changes[i][1]]=[v,a[p]];
    }
    for(const [ql,qr,qt,id]of queries){
        while(t<qt)change(t++);
        while(t>qt)change(--t);
        while(l>ql)moAdjust(f,a[--l],1);
        while(r<qr)moAdjust(f,a[r++],1);
        while(l<ql)moAdjust(f,a[l++],-1);
        while(r>qr)moAdjust(f,a[--r],-1);
        ans[id]=f.size;
    }
    return ans;
}
```

