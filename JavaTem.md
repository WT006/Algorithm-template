# Java 算法模板

## 使用约定

- 本文按知识点提供函数与数据结构；根据题目补输入输出，不把所有示例当作一个题目的解答。

- 未特别说明时，数组和图节点从 **0** 开始；每节单独说明闭区间或左闭右开区间。树状数组使用 **1** 起始下标。

- 输入须满足各节列出的前提与有效下标。Java 的和、距离及价值使用 `long`，仍需按题目核对范围；Python 整数可扩展，但算法复杂度和内存开销依然存在。

  

```java
import java.util.*;
import java.math.BigInteger;

public class Main {
    // 将选用的 static 方法、static 类粘贴到这里。
    public static void main(String[] args) throws Exception {
    }
}
```

## 内容导航

[基础算法](#基础算法) · [数据结构](#数据结构) · [搜索与图论](#搜索与图论) · [数学知识](#数学知识) · [贪心](#贪心) · [动态规划](#动态规划) · [高精度](#高精度) · [莫队](#莫队)

# 基础算法

## 快速输入

Java 的 FastScanner 放在 Main 类内；到达输入末尾时 next() 返回 null。Python 按需读取，下面演示整数输入。

```java
static class FastScanner {
    private final java.io.InputStream in;
    private final byte[] buf = new byte[1 << 16];
    private int pos, len;
    FastScanner(java.io.InputStream in) { this.in = in; }
    private int read() throws java.io.IOException {
        if (pos >= len) {
            len = in.read(buf); pos = 0;
            if (len <= 0) return -1;
        }
        return buf[pos++] & 255;
    }
    String next() throws java.io.IOException {
        StringBuilder s = new StringBuilder();
        int c;
        do { c = read(); } while (c >= 0 && c <= 32);
        if (c < 0) return null;
        while (c > 32) { s.append((char)c); c = read(); }
        return s.toString();
    }
    int nextInt() throws java.io.IOException { return Integer.parseInt(next()); }
    long nextLong() throws java.io.IOException { return Long.parseLong(next()); }
}
```

## 排序：快速排序与归并排序

原地修改数组。快速排序使用中点枢轴，最坏 O(n²)；归并排序 O(n log n)，需要 O(n) 辅助空间。

```java
static void quickSort(int[] a, int l, int r) {
    if (l >= r) return;
    int i = l - 1, j = r + 1, x = a[l + (r - l) / 2];
    while (i < j) {
        do { i++; } while (a[i] < x);
        do { j--; } while (a[j] > x);
        if (i < j) { int t = a[i]; a[i] = a[j]; a[j] = t; }
    }
    quickSort(a, l, j); quickSort(a, j + 1, r);
}
static void mergeSort(int[] a) { mergeSort(a, new int[a.length], 0, a.length); }
static void mergeSort(int[] a, int[] tmp, int l, int r) {
    if (r - l <= 1) return;
    int m = (l + r) >>> 1;
    mergeSort(a, tmp, l, m); mergeSort(a, tmp, m, r);
    int i = l, j = m, k = l;
    while (i < m && j < r) tmp[k++] = a[i] <= a[j] ? a[i++] : a[j++];
    while (i < m) tmp[k++] = a[i++];
    while (j < r) tmp[k++] = a[j++];
    System.arraycopy(tmp, l, a, l, r - l);
}
```

## 前缀和：一维与二维

查询采用左闭右开区间 [l,r) 和矩形 [x1,x2) × [y1,y2)。构建 O(n) / O(nm)，查询 O(1)。

```java
static long[] prefix(long[] a) {
    long[] s = new long[a.length + 1];
    for (int i = 0; i < a.length; i++) s[i + 1] = s[i] + a[i];
    return s; // [l,r) 的和：s[r] - s[l]
}
static long[][] prefix2D(long[][] a) {
    int n = a.length, m = n == 0 ? 0 : a[0].length;
    long[][] s = new long[n + 1][m + 1];
    for (int i = 0; i < n; i++) for (int j = 0; j < m; j++)
        s[i + 1][j + 1] = a[i][j] + s[i][j + 1] + s[i + 1][j] - s[i][j];
    return s;
}
static long rectSum(long[][] s, int x1, int y1, int x2, int y2) {
    return s[x2][y2] - s[x1][y2] - s[x2][y1] + s[x1][y1];
}
```

## 差分：一维与二维

对零数组批量区间加，最后统一还原。更新采用左闭右开区间，二维差分预留一行一列边界。

```java
static void rangeAdd(long[] d, int l, int r, long v) { d[l] += v; d[r] -= v; }
static long[] restore(long[] d) {
    long[] a = new long[d.length - 1]; long sum = 0;
    for (int i = 0; i < a.length; i++) a[i] = sum += d[i];
    return a;
}
static void rectAdd(long[][] d, int x1, int y1, int x2, int y2, long v) {
    d[x1][y1] += v; d[x2][y1] -= v; d[x1][y2] -= v; d[x2][y2] += v;
}
static long[][] restore2D(long[][] d) {
    int n = d.length - 1, m = d[0].length - 1;
    long[][] a = new long[n][m];
    for (int i = 0; i < n; i++) for (int j = 0; j < m; j++) {
        a[i][j] = d[i][j];
        if (i > 0) a[i][j] += a[i - 1][j];
        if (j > 0) a[i][j] += a[i][j - 1];
        if (i > 0 && j > 0) a[i][j] -= a[i - 1][j - 1];
    }
    return a;
}
```

## 二分：整数边界与浮点数

有序数组返回第一个 ≥ x / > x 的下标，不存在时为 n。浮点二分示例求非负数平方根，固定迭代次数。

```java
static int lowerBound(int[] a, int x) {
    int l = 0, r = a.length;
    while (l < r) { int m = (l + r) >>> 1; if (a[m] >= x) r = m; else l = m + 1; }
    return l;
}
static int upperBound(int[] a, int x) {
    int l = 0, r = a.length;
    while (l < r) { int m = (l + r) >>> 1; if (a[m] > x) r = m; else l = m + 1; }
    return l;
}
static double sqrtBinary(double x) {
    if (!Double.isFinite(x) || x < 0) throw new IllegalArgumentException();
    double l = 0, r = Math.max(1, x);
    for (int i = 0; i < 1100; i++) {
        double m = l + (r - l) / 2;
        if (m == 0 || m <= x / m) l = m; else r = m;
    }
    return l;
}
```

## 位运算

lowbit 用于正整数；统计 1 的个数时 Java 按 32 位补码，Python 版本限定非负整数。

```java
static int lowbit(int x) { return x & -x; }
static int popcount(int x) { return Integer.bitCount(x); }
```

## 区间合并

闭区间，端点相接视为重叠。返回新数组，排序 O(n log n)。

```java
static int[][] mergeIntervals(int[][] ranges) {
    int[][] a = Arrays.stream(ranges).map(int[]::clone).toArray(int[][]::new);
    Arrays.sort(a, Comparator.comparingInt(x -> x[0]));
    List<int[]> res = new ArrayList<>();
    for (int[] x : a) {
        if (res.isEmpty() || res.get(res.size() - 1)[1] < x[0]) res.add(x.clone());
        else res.get(res.size() - 1)[1] = Math.max(res.get(res.size() - 1)[1], x[1]);
    }
    return res.toArray(new int[0][]);
}
```

## 离散化

返回排序去重值及原数组对应的零基排名；排序 O(n log n)。

```java
static int[] sortedUnique(int[] a) { return Arrays.stream(a).sorted().distinct().toArray(); }
static int[] ranks(int[] a) {
    int[] values = sortedUnique(a), result = new int[a.length];
    for (int i = 0; i < a.length; i++) result[i] = Arrays.binarySearch(values, a[i]);
    return result;
}
```

# 数据结构

## 数组模拟单链表与双链表

插入返回稳定的节点编号；删除只摘链，不复用编号。单链表用 0 作头哨兵；双链表用 0、1 作左右哨兵。容量参数是最多插入次数，O(1) 更新。

```java
static class SingleList {
    int[] value, next; int idx = 1;
    SingleList(int capacity) { value = new int[capacity + 1]; next = new int[capacity + 1]; Arrays.fill(next, -1); }
    int insertAfter(int k, int x) {
        int id = idx++; value[id] = x; next[id] = next[k]; next[k] = id; return id;
    }
    void removeAfter(int k) { if (next[k] != -1) next[k] = next[next[k]]; }
}
static class DoubleList {
    int[] value, left, right; int idx = 2;
    DoubleList(int capacity) {
        value = new int[capacity + 2]; left = new int[capacity + 2]; right = new int[capacity + 2];
        right[0] = 1; left[1] = 0;
    }
    int insertAfter(int k, int x) {
        int id = idx++; value[id] = x; left[id] = k; right[id] = right[k];
        left[right[k]] = id; right[k] = id; return id;
    }
    void remove(int k) { right[left[k]] = right[k]; left[right[k]] = left[k]; }
}
```

## 栈、队列、堆与标准集合

对应 C++ STL。Java 优先用 ArrayDeque；堆操作 O(log n)，哈希查询期望 O(1)，有序集合 O(log n)。比较器使用 compare，避免相减溢出。

```java
static void collectionsExample() {
    Deque<Integer> stack = new ArrayDeque<>();
    stack.push(1); int top = stack.peek(); stack.pop();
    Deque<Integer> queue = new ArrayDeque<>();
    queue.addLast(1); int front = queue.removeFirst();
    PriorityQueue<Integer> minHeap = new PriorityQueue<>();
    PriorityQueue<Integer> maxHeap = new PriorityQueue<>(Comparator.reverseOrder());
    minHeap.add(3); int smallest = minHeap.remove();
    Map<Integer, Integer> freq = new HashMap<>();
    freq.merge(3, 1, Integer::sum);
    Set<Integer> set = new HashSet<>(); set.add(3);
    TreeSet<Integer> ordered = new TreeSet<>(); ordered.add(3);
    Integer ceil = ordered.ceiling(2), floor = ordered.floor(4);
}
```

## 单调栈与单调队列

返回左侧最近严格较小元素的下标，以及长度 k 的滑动窗口最小值。均为 O(n)，要求 1 ≤ k ≤ n。

```java
static int[] previousLess(int[] a) {
    int[] res = new int[a.length], stack = new int[a.length]; int top = 0;
    for (int i = 0; i < a.length; i++) {
        while (top > 0 && a[stack[top - 1]] >= a[i]) top--;
        res[i] = top == 0 ? -1 : stack[top - 1]; stack[top++] = i;
    }
    return res;
}
static int[] windowMin(int[] a, int k) {
    if (k < 1 || k > a.length) throw new IllegalArgumentException();
    Deque<Integer> q = new ArrayDeque<>(); int[] res = new int[a.length - k + 1];
    for (int i = 0; i < a.length; i++) {
        while (!q.isEmpty() && q.peekFirst() <= i - k) q.removeFirst();
        while (!q.isEmpty() && a[q.peekLast()] >= a[i]) q.removeLast();
        q.addLast(i); if (i >= k - 1) res[i - k + 1] = a[q.peekFirst()];
    }
    return res;
}
```

## KMP 字符串匹配

返回所有匹配起点，支持重叠；空模式定义为匹配每个边界。O(n+m)。Java 下标按 UTF-16 char 计数。

```java
static List<Integer> kmp(String text, String pattern) {
    int m = pattern.length(); List<Integer> res = new ArrayList<>();
    if (m == 0) { for (int i = 0; i <= text.length(); i++) res.add(i); return res; }
    int[] pi = new int[m];
    for (int i = 1, j = 0; i < m; i++) {
        while (j > 0 && pattern.charAt(i) != pattern.charAt(j)) j = pi[j - 1];
        if (pattern.charAt(i) == pattern.charAt(j)) j++;
        pi[i] = j;
    }
    for (int i = 0, j = 0; i < text.length(); i++) {
        while (j > 0 && text.charAt(i) != pattern.charAt(j)) j = pi[j - 1];
        if (text.charAt(i) == pattern.charAt(j)) j++;
        if (j == m) { res.add(i - m + 1); j = pi[j - 1]; }
    }
    return res;
}
```

## Trie 字典树

支持插入和统计完整单词出现次数，O(单词长度)。Java 版本限定小写英文字母；Python 使用字典存边。

```java
static class Trie {
    static class Node { Node[] next = new Node[26]; int count; }
    Node root = new Node();
    void insert(String s) {
        Node p = root;
        for (char c : s.toCharArray()) {
            int x = c - 'a'; if (p.next[x] == null) p.next[x] = new Node(); p = p.next[x];
        }
        p.count++;
    }
    int count(String s) {
        Node p = root;
        for (char c : s.toCharArray()) { p = p.next[c - 'a']; if (p == null) return 0; }
        return p.count;
    }
}
```

## 并查集

路径压缩与按大小合并，均摊 O(α(n))。重复合并不改变集合大小。

```java
static class DSU {
    int[] parent, size;
    DSU(int n) {
        parent = new int[n]; size = new int[n];
        for (int i = 0; i < n; i++) { parent[i] = i; size[i] = 1; }
    }
    int find(int x) {
        while (parent[x] != x) { parent[x] = parent[parent[x]]; x = parent[x]; }
        return x;
    }
    boolean union(int a, int b) {
        a = find(a); b = find(b); if (a == b) return false;
        if (size[a] < size[b]) { int t = a; a = b; b = t; }
        parent[b] = a; size[a] += size[b]; return true;
    }
}
```

## 可修改、可删除的编号堆

编号由调用方提供且当前唯一；支持插入、按编号修改、删除、取最小值，O(log n)。普通堆与标准库写法见前文。

```java
static class IndexedHeap {
    List<long[]> heap = new ArrayList<>(); // {值, 编号}
    Map<Integer, Integer> pos = new HashMap<>();
    void swap(int i, int j) {
        long[] t = heap.get(i); heap.set(i, heap.get(j)); heap.set(j, t);
        pos.put((int)heap.get(i)[1], i); pos.put((int)heap.get(j)[1], j);
    }
    int up(int i) {
        while (i > 0) { int p = (i - 1) / 2; if (heap.get(p)[0] <= heap.get(i)[0]) break; swap(i, p); i = p; }
        return i;
    }
    void down(int i) {
        while (2 * i + 1 < heap.size()) {
            int j = 2 * i + 1;
            if (j + 1 < heap.size() && heap.get(j + 1)[0] < heap.get(j)[0]) j++;
            if (heap.get(i)[0] <= heap.get(j)[0]) break; swap(i, j); i = j;
        }
    }
    void insert(int id, long value) {
        if (pos.containsKey(id)) throw new IllegalArgumentException("duplicate id");
        pos.put(id, heap.size()); heap.add(new long[]{value, id}); up(heap.size() - 1);
    }
    void change(int id, long value) { int i = pos.get(id); heap.get(i)[0] = value; down(up(i)); }
    void remove(int id) {
        int i = pos.get(id), last = heap.size() - 1;
        swap(i, last); heap.remove(last); pos.remove(id);
        if (i < heap.size()) down(up(i));
    }
    long[] peek() { return heap.get(0).clone(); }
}
```

## 哈希：开放寻址与链表法

整数集合，仅插入和查询。开放寻址表容量必须大于不同元素数；使用占用标记，不占用某个整数作空值。链表法允许键碰撞。

```java
static class OpenHash {
    int[] keys; boolean[] used;
    OpenHash(int capacity) { keys = new int[capacity]; used = new boolean[capacity]; }
    int locate(int x) {
        int k = Math.floorMod(x, keys.length), start = k;
        while (used[k] && keys[k] != x) {
            k = (k + 1) % keys.length;
            if (k == start) throw new IllegalStateException("table full");
        }
        return k;
    }
    void add(int x) { int k = locate(x); keys[k] = x; used[k] = true; }
    boolean contains(int x) { int k = locate(x); return used[k]; }
}
static class ChainHash {
    List<List<Integer>> buckets = new ArrayList<>();
    ChainHash(int capacity) { for (int i = 0; i < capacity; i++) buckets.add(new ArrayList<>()); }
    void add(int x) { List<Integer> b = buckets.get(Math.floorMod(x, buckets.size())); if (!b.contains(x)) b.add(x); }
    boolean contains(int x) { return buckets.get(Math.floorMod(x, buckets.size())).contains(x); }
}
```

## 字符串前缀哈希

查询 [l,r) 的哈希。固定 base=131、mod=1000000007；哈希相同可能碰撞，需要精确结果时再核对原字符串。O(n) 构建、O(1) 查询。Java 与 Python 此处均以 BMP 字符作为一致的对照输入。

```java
static class StringHash {
    static final long MOD = 1_000_000_007L, BASE = 131;
    long[] h, power;
    StringHash(String s) {
        int n = s.length(); h = new long[n + 1]; power = new long[n + 1]; power[0] = 1;
        for (int i = 0; i < n; i++) {
            h[i + 1] = (h[i] * BASE + s.charAt(i) + 1) % MOD;
            power[i + 1] = power[i] * BASE % MOD;
        }
    }
    long get(int l, int r) { return (h[r] - h[l] * power[r - l] % MOD + MOD) % MOD; }
}
```

## 二叉树遍历

显式节点表示一般二叉树，不要求满二叉树。返回前序、中序、后序三种序列；迭代实现，O(n)。

```java
static class TreeNode {
    int value; TreeNode left, right;
    TreeNode(int value) { this.value = value; }
}
static List<List<Integer>> traversals(TreeNode root) {
    List<Integer> pre = new ArrayList<>(), in = new ArrayList<>(), post = new ArrayList<>();
    Deque<TreeNode> stack = new ArrayDeque<>();
    if (root != null) stack.push(root);
    while (!stack.isEmpty()) {
        TreeNode u = stack.pop(); pre.add(u.value);
        if (u.right != null) stack.push(u.right); if (u.left != null) stack.push(u.left);
    }
    TreeNode cur = root;
    while (cur != null || !stack.isEmpty()) {
        while (cur != null) { stack.push(cur); cur = cur.left; }
        cur = stack.pop(); in.add(cur.value); cur = cur.right;
    }
    if (root != null) stack.push(root);
    while (!stack.isEmpty()) {
        TreeNode u = stack.pop(); post.add(u.value);
        if (u.left != null) stack.push(u.left); if (u.right != null) stack.push(u.right);
    }
    Collections.reverse(post); return Arrays.asList(pre, in, post);
}
```

## 哈夫曼树与合并代价

非负权值；每次合并最小两项，返回带权路径长度。空集合或单节点代价为 0，O(n log n)。贪心中的合并果子使用同一算法。

```java
static long huffmanCost(long[] weights) {
    PriorityQueue<Long> q = new PriorityQueue<>();
    for (long w : weights) q.add(w);
    long ans = 0;
    while (q.size() > 1) { long sum = q.remove() + q.remove(); ans += sum; q.add(sum); }
    return ans;
}
```

## 树状数组：点加、区间加与区间和

本节使用一基下标。prefix(r) 查询 [1,r]；rangeAdd(l,r,v) 更新闭区间 [l,r]。单次操作 O(log n)，构造后为零数组。

```java
static class Fenwick {
    int n; long[] tree;
    Fenwick(int n) { this.n = n; tree = new long[n + 1]; }
    void add(int i, long v) { if (i <= 0) throw new IllegalArgumentException(); for (; i <= n; i += i & -i) tree[i] += v; }
    long prefix(int i) { long s = 0; for (; i > 0; i -= i & -i) s += tree[i]; return s; }
    long rangeSum(int l, int r) { return prefix(r) - prefix(l - 1); }
}
static class RangeFenwick {
    Fenwick b1, b2;
    RangeFenwick(int n) { b1 = new Fenwick(n); b2 = new Fenwick(n); }
    void rangeAdd(int l, int r, long v) {
        b1.add(l, v); b1.add(r + 1, -v);
        b2.add(l, v * (l - 1)); b2.add(r + 1, -v * r);
    }
    long prefix(int r) { return b1.prefix(r) * r - b2.prefix(r); }
    long rangeSum(int l, int r) { return prefix(r) - prefix(l - 1); }
    long point(int i) { return rangeSum(i, i); }
}
```

## 线段树：懒标记区间加、区间和

零基闭区间 [l,r]，要求原数组非空；更新、查询 O(log n)。

```java
static class SegmentTree {
    int n; long[] sum, lazy;
    SegmentTree(long[] a) {
        n = a.length; if (n == 0) throw new IllegalArgumentException();
        sum = new long[4 * n]; lazy = new long[4 * n]; build(1, 0, n - 1, a);
    }
    void build(int u, int l, int r, long[] a) {
        if (l == r) { sum[u] = a[l]; return; }
        int m = (l + r) >>> 1; build(u * 2, l, m, a); build(u * 2 + 1, m + 1, r, a); pull(u);
    }
    void pull(int u) { sum[u] = sum[u * 2] + sum[u * 2 + 1]; }
    void apply(int u, int l, int r, long v) { sum[u] += (r - l + 1L) * v; lazy[u] += v; }
    void push(int u, int l, int r) {
        int m = (l + r) >>> 1;
        apply(u * 2, l, m, lazy[u]); apply(u * 2 + 1, m + 1, r, lazy[u]); lazy[u] = 0;
    }
    void add(int ql, int qr, long v) { add(1, 0, n - 1, ql, qr, v); }
    void add(int u, int l, int r, int ql, int qr, long v) {
        if (ql <= l && r <= qr) { apply(u, l, r, v); return; }
        push(u, l, r); int m = (l + r) >>> 1;
        if (ql <= m) add(u * 2, l, m, ql, qr, v);
        if (qr > m) add(u * 2 + 1, m + 1, r, ql, qr, v);
        pull(u);
    }
    long query(int ql, int qr) { return query(1, 0, n - 1, ql, qr); }
    long query(int u, int l, int r, int ql, int qr) {
        if (ql <= l && r <= qr) return sum[u];
        push(u, l, r); int m = (l + r) >>> 1; long res = 0;
        if (ql <= m) res += query(u * 2, l, m, ql, qr);
        if (qr > m) res += query(u * 2 + 1, m + 1, r, ql, qr);
        return res;
    }
}
```

## 树形 DP：最大权独立集

输入非空无向树与节点权值；相邻节点不能同时选，可全不选。迭代建树、逆序转移，O(n)。

```java
static long treeIndependentSet(List<Integer>[] g, long[] weight) {
    int n = g.length; int[] parent = new int[n], order = new int[n]; Arrays.fill(parent, -2);
    parent[0] = -1; order[0] = 0; int count = 1;
    for (int i = 0; i < count; i++) {
        int u = order[i];
        for (int v : g[u]) if (parent[v] == -2) { parent[v] = u; order[count++] = v; }
    }
    long[][] dp = new long[n][2];
    for (int i = n - 1; i >= 0; i--) {
        int u = order[i]; dp[u][1] += weight[u];
        if (parent[u] >= 0) {
            int p = parent[u]; dp[p][0] += Math.max(dp[u][0], dp[u][1]); dp[p][1] += dp[u][0];
        }
    }
    return Math.max(dp[0][0], dp[0][1]);
}
```

# 搜索与图论

## DFS：全排列与 N 皇后

返回全部解，内存随解数量增长；递归仅用于搜索深度较小的题。全排列 O(n·n!)；N 皇后按列与对角线剪枝。

```java
static List<int[]> permutations(int n) {
    List<int[]> ans = new ArrayList<>(); permDfs(0, new int[n], new boolean[n], ans); return ans;
}
static void permDfs(int d, int[] path, boolean[] used, List<int[]> ans) {
    if (d == path.length) { ans.add(path.clone()); return; }
    for (int x = 0; x < path.length; x++) if (!used[x]) {
        used[x] = true; path[d] = x; permDfs(d + 1, path, used, ans); used[x] = false;
    }
}
static List<int[]> nQueens(int n) {
    List<int[]> ans = new ArrayList<>();
    queenDfs(0, new int[n], new boolean[n], new boolean[2 * n + 1], new boolean[2 * n + 1], ans);
    return ans;
}
static void queenDfs(int r, int[] path, boolean[] col, boolean[] d1, boolean[] d2, List<int[]> ans) {
    int n = path.length; if (r == n) { ans.add(path.clone()); return; }
    for (int c = 0; c < n; c++) if (!col[c] && !d1[r + c] && !d2[r - c + n]) {
        col[c] = d1[r + c] = d2[r - c + n] = true; path[r] = c;
        queenDfs(r + 1, path, col, d1, d2, ans);
        col[c] = d1[r + c] = d2[r - c + n] = false;
    }
}
```

## 树的重心

对应原模板“树的中心”的实际算法：返回删除单个节点后最大连通块大小的最小值；输入非空无向树。O(n)。

```java
static int centroidBalance(List<Integer>[] g) {
    int n = g.length; int[] parent = new int[n], order = new int[n], size = new int[n];
    Arrays.fill(parent, -2); parent[0] = -1; order[0] = 0; int count = 1;
    for (int i = 0; i < count; i++) for (int v : g[order[i]]) if (parent[v] == -2) {
        parent[v] = order[i]; order[count++] = v;
    }
    int best = n;
    for (int i = n - 1; i >= 0; i--) {
        int u = order[i], largest = 0; size[u]++;
        for (int v : g[u]) if (parent[v] == u) { size[u] += size[v]; largest = Math.max(largest, size[v]); }
        best = Math.min(best, Math.max(largest, n - size[u]));
    }
    return best;
}
```

## BFS：图的层次与迷宫

无权图，返回源点到所有点的距离，不可达为 -1。迷宫用 0 表示可走、1 表示障碍，起终点分别为左上角和右下角。O(V+E) / O(nm)。

```java
static int[] bfs(List<Integer>[] g, int source) {
    int[] d = new int[g.length]; Arrays.fill(d, -1); d[source] = 0;
    Deque<Integer> q = new ArrayDeque<>(); q.add(source);
    while (!q.isEmpty()) { int u = q.remove(); for (int v : g[u]) if (d[v] == -1) { d[v] = d[u] + 1; q.add(v); } }
    return d;
}
static int mazeDistance(int[][] a) {
    int n = a.length; if (n == 0 || a[0].length == 0) return -1;
    int m = a[0].length; if (a[0][0] != 0 || a[n - 1][m - 1] != 0) return -1;
    int[][] d = new int[n][m]; for (int[] row : d) Arrays.fill(row, -1);
    int[] dx = {-1, 0, 1, 0}, dy = {0, 1, 0, -1};
    Deque<int[]> q = new ArrayDeque<>(); q.add(new int[]{0, 0}); d[0][0] = 0;
    while (!q.isEmpty()) {
        int[] u = q.remove();
        for (int k = 0; k < 4; k++) {
            int x = u[0] + dx[k], y = u[1] + dy[k];
            if (x >= 0 && x < n && y >= 0 && y < m && a[x][y] == 0 && d[x][y] == -1) {
                d[x][y] = d[u[0]][u[1]] + 1; q.add(new int[]{x, y});
            }
        }
    }
    return d[n - 1][m - 1];
}
```

## Dijkstra：朴素与堆优化

边权非负。朴素版矩阵无边设为 INF，O(n²)；堆版邻接项为 {终点,权重}，O((V+E) log V)。不可达为 Java Long.MAX_VALUE/4、Python inf。

```java
static long[] dijkstraDense(long[][] g, int source) {
    long inf = Long.MAX_VALUE / 4; int n = g.length;
    long[] d = new long[n]; boolean[] used = new boolean[n]; Arrays.fill(d, inf); d[source] = 0;
    for (int i = 0; i < n; i++) {
        int u = -1;
        for (int j = 0; j < n; j++) if (!used[j] && (u == -1 || d[j] < d[u])) u = j;
        if (u == -1 || d[u] == inf) break; used[u] = true;
        for (int v = 0; v < n; v++) if (g[u][v] != inf) d[v] = Math.min(d[v], d[u] + g[u][v]);
    }
    return d;
}
static long[] dijkstra(List<int[]>[] g, int source) {
    long[] d = new long[g.length]; Arrays.fill(d, Long.MAX_VALUE / 4); d[source] = 0;
    PriorityQueue<long[]> q = new PriorityQueue<>(Comparator.comparingLong(x -> x[0])); q.add(new long[]{0, source});
    while (!q.isEmpty()) {
        long[] cur = q.remove(); int u = (int)cur[1]; if (cur[0] != d[u]) continue;
        for (int[] e : g[u]) if (d[e[0]] > d[u] + e[1]) {
            d[e[0]] = d[u] + e[1]; q.add(new long[]{d[e[0]], e[0]});
        }
    }
    return d;
}
```

## Bellman-Ford：最多 k 条边

边表每项为 {起点,终点,权重}。每轮读取上一轮副本，允许负权与负环，但限制最多 k 条边；不可达为 INF / inf。O(kE)。

```java
static long[] bellmanFord(int n, int[][] edges, int source, int k) {
    long inf = Long.MAX_VALUE / 4; long[] d = new long[n]; Arrays.fill(d, inf); d[source] = 0;
    for (int i = 0; i < k; i++) {
        long[] old = d.clone();
        for (int[] e : edges) if (old[e[0]] != inf) d[e[1]] = Math.min(d[e[1]], old[e[0]] + e[2]);
    }
    return d;
}
```

## SPFA

允许负权；检测到从源点可达的负环时抛出异常。邻接项为 {终点,权重}。最坏 O(VE)，不可达为 INF / inf。

```java
static long[] spfa(List<int[]>[] g, int source) {
    int n = g.length; long[] d = new long[n]; int[] count = new int[n]; boolean[] inQueue = new boolean[n];
    Arrays.fill(d, Long.MAX_VALUE / 4); d[source] = 0;
    Deque<Integer> q = new ArrayDeque<>(); q.add(source); inQueue[source] = true;
    while (!q.isEmpty()) {
        int u = q.remove(); inQueue[u] = false;
        for (int[] e : g[u]) {
            int v = e[0];
            if (d[v] > d[u] + e[1]) {
                d[v] = d[u] + e[1]; count[v] = count[u] + 1;
                if (count[v] >= n) throw new IllegalArgumentException("reachable negative cycle");
                if (!inQueue[v]) { inQueue[v] = true; q.add(v); }
            }
        }
    }
    return d;
}
```

## Floyd 全源最短路

允许负权，要求没有负环；矩阵对角线为 0，无边为 INF / inf，重边建图取最小值。返回副本，O(n³)。

```java
static long[][] floyd(long[][] g) {
    long inf = Long.MAX_VALUE / 4; int n = g.length;
    long[][] d = Arrays.stream(g).map(long[]::clone).toArray(long[][]::new);
    for (int k = 0; k < n; k++) for (int i = 0; i < n; i++) if (d[i][k] != inf)
        for (int j = 0; j < n; j++) if (d[k][j] != inf) d[i][j] = Math.min(d[i][j], d[i][k] + d[k][j]);
    return d;
}
```

## 最小生成树：Prim 与 Kruskal

无向图，允许负权与重边。Prim 读矩阵，无边为 INF / inf，O(n²)；Kruskal 读 {u,v,w} 边表，依赖前文 DSU，O(E log E)。非连通返回 Java null、Python None，空图返回 0。

```java
static Long prim(long[][] g) {
    int n = g.length; long inf = Long.MAX_VALUE / 4, ans = 0;
    if (n == 0) return 0L;
    long[] d = new long[n]; boolean[] used = new boolean[n]; Arrays.fill(d, inf); d[0] = 0;
    for (int i = 0; i < n; i++) {
        int u = -1;
        for (int j = 0; j < n; j++) if (!used[j] && (u == -1 || d[j] < d[u])) u = j;
        if (d[u] == inf) return null; ans += d[u]; used[u] = true;
        for (int v = 0; v < n; v++) if (!used[v]) d[v] = Math.min(d[v], g[u][v]);
    }
    return ans;
}
static Long kruskal(int n, int[][] edges) {
    int[][] sorted = edges.clone(); Arrays.sort(sorted, Comparator.comparingInt(e -> e[2]));
    DSU dsu = new DSU(n); long ans = 0; int count = 0;
    for (int[] e : sorted) if (dsu.union(e[0], e[1])) { ans += e[2]; count++; }
    return n == 0 || count == n - 1 ? Long.valueOf(ans) : null;
}
```

## 二分图：染色与匈牙利算法

染色读取无向邻接表，O(V+E)。匹配输入仅为左侧点的邻接表，邻接值是右侧编号，O(VE)；使用迭代增广，避免长交替路的递归栈限制。

```java
static boolean isBipartite(List<Integer>[] g) {
    int[] color = new int[g.length]; Deque<Integer> q = new ArrayDeque<>();
    for (int s = 0; s < g.length; s++) if (color[s] == 0) {
        color[s] = 1; q.add(s);
        while (!q.isEmpty()) {
            int u = q.remove();
            for (int v : g[u]) {
                if (color[v] == color[u]) return false;
                if (color[v] == 0) { color[v] = 3 - color[u]; q.add(v); }
            }
        }
    }
    return true;
}
static int maxMatching(List<Integer>[] g, int rightCount) {
    int[] left = new int[g.length], right = new int[rightCount];
    Arrays.fill(left, -1); Arrays.fill(right, -1); int ans = 0;
    for (int s = 0; s < g.length; s++) {
        int[] prevRight = new int[rightCount]; Arrays.fill(prevRight, -1);
        boolean[] seenLeft = new boolean[g.length]; seenLeft[s] = true;
        Deque<Integer> q = new ArrayDeque<>(); q.add(s); int end = -1;
        while (!q.isEmpty() && end == -1) {
            int u = q.remove();
            for (int v : g[u]) if (prevRight[v] == -1) {
                prevRight[v] = u;
                if (right[v] == -1) { end = v; break; }
                if (!seenLeft[right[v]]) { seenLeft[right[v]] = true; q.add(right[v]); }
            }
        }
        if (end >= 0) {
            ans++;
            while (end >= 0) {
                int u = prevRight[end], old = left[u]; left[u] = end; right[end] = u; end = old;
            }
        }
    }
    return ans;
}
```

## 拓扑排序

有向图；存在环返回 null / None，否则返回一个拓扑序，O(V+E)。

```java
static int[] topologicalSort(List<Integer>[] g) {
    int n = g.length; int[] indegree = new int[n], order = new int[n];
    for (List<Integer> row : g) for (int v : row) indegree[v]++;
    Deque<Integer> q = new ArrayDeque<>(); for (int i = 0; i < n; i++) if (indegree[i] == 0) q.add(i);
    int count = 0;
    while (!q.isEmpty()) { int u = q.remove(); order[count++] = u; for (int v : g[u]) if (--indegree[v] == 0) q.add(v); }
    return count == n ? order : null;
}
```

# 数学知识

## 质数：试除、分解与筛法

factorize 输入正整数，返回质因子到指数的映射；筛法统一包含上界 n。试除 O(√n)，埃氏筛 O(n log log n)，线性筛 O(n)。

```java
static boolean isPrime(int n) {
    if (n < 2) return false;
    for (int i = 2; i <= n / i; i++) if (n % i == 0) return false;
    return true;
}
static Map<Integer, Integer> factorize(int n) {
    if (n < 1) throw new IllegalArgumentException();
    Map<Integer, Integer> factors = new LinkedHashMap<>();
    for (int p = 2; p <= n / p; p++) if (n % p == 0) {
        int count = 0; while (n % p == 0) { n /= p; count++; } factors.put(p, count);
    }
    if (n > 1) factors.put(n, 1); return factors;
}
static List<Integer> sieve(int n) {
    boolean[] composite = new boolean[n + 1]; List<Integer> primes = new ArrayList<>();
    for (int i = 2; i <= n; i++) if (!composite[i]) {
        primes.add(i);
        if ((long)i * i <= n) for (long j = (long)i * i; j <= n; j += i) composite[(int)j] = true;
    }
    return primes;
}
static List<Integer> linearSieve(int n) {
    boolean[] composite = new boolean[n + 1]; List<Integer> primes = new ArrayList<>();
    for (int i = 2; i <= n; i++) {
        if (!composite[i]) primes.add(i);
        for (int p : primes) {
            if (p > n / i) break; composite[i * p] = true;
            if (i % p == 0) break;
        }
    }
    return primes;
}
```

## 约数、约数个数与约数之和

divisors 输入正整数，O(√n) 枚举后排序。个数与和从质因子指数表计算，依赖前文 factorize；模数由调用方传入。Java 要求 1 ≤ mod ≤ 2147483647，使中间模乘可用 long。

```java
static List<Integer> divisors(int n) {
    if (n < 1) throw new IllegalArgumentException();
    List<Integer> ans = new ArrayList<>();
    for (int i = 1; i <= n / i; i++) if (n % i == 0) { ans.add(i); if (i != n / i) ans.add(n / i); }
    Collections.sort(ans); return ans;
}
static long divisorCount(Map<Integer, Integer> factors, int mod) {
    long ans = 1 % mod; for (int e : factors.values()) ans = ans * (e + 1L) % mod; return ans;
}
static long divisorSum(Map<Integer, Integer> factors, int mod) {
    long ans = 1 % mod;
    for (Map.Entry<Integer, Integer> entry : factors.entrySet()) {
        long term = 1 % mod;
        for (int i = 0; i < entry.getValue(); i++) term = (term * entry.getKey() + 1) % mod;
        ans = ans * term % mod;
    }
    return ans;
}
```

## 欧几里得、扩展欧几里得与线性同余方程

输入非负整数且不同时为 0；扩展算法返回 (g,x,y)，满足 ax+by=g。解 ax≡b (mod m) 时要求 m>0，无解返回 null / None。Java 的中间系数乘法须在 long 范围内。

```java
static long gcd(long a, long b) { while (b != 0) { long t = a % b; a = b; b = t; } return a; }
static long[] exgcd(long a, long b) {
    if (b == 0) return new long[]{a, 1, 0};
    long[] r = exgcd(b, a % b); return new long[]{r[0], r[2], r[1] - a / b * r[2]};
}
static Long solveCongruence(long a, long b, long m) {
    long[] r = exgcd(a, m); if (b % r[0] != 0) return null;
    long period = m / r[0]; return Math.floorMod(r[1] * (b / r[0]), period);
}
```

## 快速幂

b≥0，mod>0；O(log b)。Java 限定 mod≤2147483647，避免 long 模乘溢出。mod=1 时正确返回 0。

```java
static long modPow(long a, long b, int mod) {
    a = Math.floorMod(a, (long)mod); long res = 1L % mod;
    while (b > 0) { if ((b & 1) != 0) res = res * a % mod; a = a * a % mod; b >>= 1; }
    return res;
}
```

## 高斯消元

n 个方程、n 个未知数，增广矩阵 n×(n+1)，直接修改矩阵。返回 0=唯一解、1=无穷多解、2=无解；唯一解存最后一列。O(n³)，按 eps=1e-9 判断零。

```java
static int gauss(double[][] a) {
    int n = a.length, r = 0; double eps = 1e-9;
    for (int c = 0; c < n; c++) {
        int t = r; for (int i = r; i < n; i++) if (Math.abs(a[i][c]) > Math.abs(a[t][c])) t = i;
        if (Math.abs(a[t][c]) < eps) continue;
        double[] tmp = a[t]; a[t] = a[r]; a[r] = tmp;
        for (int j = n; j >= c; j--) a[r][j] /= a[r][c];
        for (int i = r + 1; i < n; i++) if (Math.abs(a[i][c]) > eps)
            for (int j = n; j >= c; j--) a[i][j] -= a[r][j] * a[i][c];
        r++;
    }
    if (r < n) { for (int i = r; i < n; i++) if (Math.abs(a[i][n]) > eps) return 2; return 1; }
    for (int i = n - 1; i >= 0; i--) for (int j = i + 1; j < n; j++) a[i][n] -= a[i][j] * a[j][n];
    return 0;
}
```

## 组合数：递推、阶乘逆元、Lucas 与精确值

递推适用任意正模数。阶乘逆元要求 p 为质数且 n<p；Lucas 要求 p 为质数，单次小组合 O(p)，适合较小 p。Java p≤2147483647，依赖前文 modPow；精确组合数使用 BigInteger / 整数。

```java
static long[][] combinationTable(int n, int mod) {
    long[][] c = new long[n + 1][n + 1];
    for (int i = 0; i <= n; i++) { c[i][0] = 1L % mod; for (int j = 1; j <= i; j++) c[i][j] = (c[i - 1][j - 1] + c[i - 1][j]) % mod; }
    return c;
}
static class CombPrime {
    long[] fact, invFact; int p;
    CombPrime(int n, int p) {
        if (n >= p || p < 2) throw new IllegalArgumentException(); this.p = p;
        fact = new long[n + 1]; invFact = new long[n + 1]; fact[0] = 1;
        for (int i = 1; i <= n; i++) fact[i] = fact[i - 1] * i % p;
        invFact[n] = modPow(fact[n], p - 2, p);
        for (int i = n; i > 0; i--) invFact[i - 1] = invFact[i] * i % p;
    }
    long choose(int n, int k) { return k < 0 || k > n ? 0 : fact[n] * invFact[k] % p * invFact[n - k] % p; }
}
static long smallComb(long n, long k, int p) {
    if (k < 0 || k > n) return 0;
    k = Math.min(k, n - k); long a = 1, b = 1;
    for (long i = 1; i <= k; i++) { a = a * (n - i + 1) % p; b = b * i % p; }
    return a * modPow(b, p - 2, p) % p;
}
static long lucas(long n, long k, int p) {
    if (p < 2) throw new IllegalArgumentException(); long ans = 1;
    while (n > 0 || k > 0) { ans = ans * smallComb(n % p, k % p, p) % p; n /= p; k /= p; }
    return ans;
}
static BigInteger exactComb(int n, int k) {
    if (k < 0 || k > n) return BigInteger.ZERO;
    BigInteger ans = BigInteger.ONE; k = Math.min(k, n - k);
    for (int i = 1; i <= k; i++) ans = ans.multiply(BigInteger.valueOf(n - i + 1L)).divide(BigInteger.valueOf(i));
    return ans;
}
```

## 卡特兰数

n≥0，返回精确值 C(2n,n)/(n+1)，依赖 exactComb / exact_comb；模任意正数可在得到精确值后取模。

```java
static BigInteger catalan(int n) { return exactComb(2 * n, n).divide(BigInteger.valueOf(n + 1L)); }
```

# 贪心

## 合并果子、区间覆盖、分组与选点

合并果子调用前文 huffmanCost / huffman_cost。其余均为闭区间，O(n log n)。区间覆盖要求 start<end，无法覆盖返回 -1；分组中端点相接算冲突，选点中一个端点可以覆盖多个区间。

```java
static int coverIntervals(int start, int end, int[][] ranges) {
    int[][] a = ranges.clone(); Arrays.sort(a, Comparator.comparingInt(x -> x[0]));
    int i = 0, count = 0;
    while (start < end) {
        int next = start;
        while (i < a.length && a[i][0] <= start) { next = Math.max(next, a[i][1]); i++; }
        if (next == start) return -1; start = next; count++;
    }
    return count;
}
static int groupIntervals(int[][] ranges) {
    int[][] a = ranges.clone(); Arrays.sort(a, Comparator.comparingInt(x -> x[0]));
    PriorityQueue<Integer> ends = new PriorityQueue<>();
    for (int[] r : a) { if (!ends.isEmpty() && ends.peek() < r[0]) ends.remove(); ends.add(r[1]); }
    return ends.size();
}
static int selectPoints(int[][] ranges) {
    int[][] a = ranges.clone(); Arrays.sort(a, Comparator.comparingInt(x -> x[1]));
    long point = Long.MIN_VALUE; int count = 0;
    for (int[] r : a) if (point < r[0]) { count++; point = r[1]; }
    return count;
}
```

# 动态规划

## 背包：01、完全、多重与分组

容量至多为 m，可不选物品。物品体积为正整数，价值允许负数。01 / 完全背包 O(nm)，多重背包二进制拆分后 O(mΣlog(s+1))，分组背包 O(mΣ组大小)。Java 价值与状态用 long。

```java
static long knapsack01(int m, int[] volume, long[] value) {
    long[] f = new long[m + 1];
    for (int i = 0; i < volume.length; i++) for (int j = m; j >= volume[i]; j--) f[j] = Math.max(f[j], f[j - volume[i]] + value[i]);
    return f[m];
}
static long knapsackComplete(int m, int[] volume, long[] value) {
    long[] f = new long[m + 1];
    for (int i = 0; i < volume.length; i++) for (int j = volume[i]; j <= m; j++) f[j] = Math.max(f[j], f[j - volume[i]] + value[i]);
    return f[m];
}
static long knapsackBounded(int m, int[] volume, long[] value, int[] count) {
    long[] f = new long[m + 1];
    for (int i = 0; i < volume.length; i++) {
        int remaining = Math.min(count[i], m / volume[i]);
        for (long k = 1; remaining > 0; k *= 2) {
            int take = (int)Math.min(k, remaining); remaining -= take;
            int v = volume[i] * take; long w = value[i] * take;
            for (int j = m; j >= v; j--) f[j] = Math.max(f[j], f[j - v] + w);
        }
    }
    return f[m];
}
static long knapsackGrouped(int m, int[][][] groups) { // 每组物品为 {体积, 价值}
    long[] f = new long[m + 1];
    for (int[][] group : groups) {
        long[] old = f.clone();
        for (int j = 0; j <= m; j++) for (int[] item : group) if (j >= item[0]) f[j] = Math.max(f[j], old[j - item[0]] + item[1]);
    }
    return f[m];
}
```

## 线性 DP：数字三角形

第 i 行恰有 i+1 项；求从顶到底的最大路径和，允许负数。自底向上，O(n²) 时间、O(n) 额外空间；空三角形返回 0。

```java
static long triangleMax(long[][] a) {
    if (a.length == 0) return 0;
    long[] f = a[a.length - 1].clone();
    for (int i = a.length - 2; i >= 0; i--) for (int j = 0; j <= i; j++) f[j] = a[i][j] + Math.max(f[j], f[j + 1]);
    return f[0];
}
```

## 最长上升子序列

严格上升；lis 返回一条正向序列，O(n²)，lisLength 返回长度，O(n log n)。空数组返回空序列或 0。

```java
static int[] lis(int[] a) {
    int n = a.length; if (n == 0) return new int[0];
    int[] f = new int[n], prev = new int[n]; Arrays.fill(f, 1); Arrays.fill(prev, -1); int best = 0;
    for (int i = 0; i < n; i++) {
        for (int j = 0; j < i; j++) if (a[j] < a[i] && f[j] + 1 > f[i]) { f[i] = f[j] + 1; prev[i] = j; }
        if (f[i] > f[best]) best = i;
    }
    int[] ans = new int[f[best]];
    for (int i = ans.length - 1; i >= 0; i--) { ans[i] = a[best]; best = prev[best]; }
    return ans;
}
static int lisLength(int[] a) {
    int[] tails = new int[a.length]; int size = 0;
    for (int x : a) {
        int l = 0, r = size;
        while (l < r) { int m = (l + r) >>> 1; if (tails[m] >= x) r = m; else l = m + 1; }
        tails[l] = x; if (l == size) size++;
    }
    return size;
}
```

## 最长公共子序列

返回长度，O(nm) 时间、O(m) 空间；Java 按 UTF-16 char 比较。

```java
static int lcsLength(String a, String b) {
    int[] f = new int[b.length() + 1];
    for (int i = 0; i < a.length(); i++) {
        int diagonal = 0;
        for (int j = 1; j <= b.length(); j++) {
            int old = f[j];
            f[j] = a.charAt(i) == b.charAt(j - 1) ? diagonal + 1 : Math.max(f[j], f[j - 1]);
            diagonal = old;
        }
    }
    return f[b.length()];
}
```

## 区间 DP：石子合并

每次只能合并相邻两堆，求最小总代价，线性排列；O(n³) 时间、O(n²) 空间。

```java
static long stoneMerge(long[] a) {
    int n = a.length; if (n == 0) return 0;
    long[] s = new long[n + 1]; for (int i = 0; i < n; i++) s[i + 1] = s[i] + a[i];
    long[][] f = new long[n][n];
    for (int len = 2; len <= n; len++) for (int l = 0; l + len <= n; l++) {
        int r = l + len - 1; f[l][r] = Long.MAX_VALUE / 4;
        for (int k = l; k < r; k++) f[l][r] = Math.min(f[l][r], f[l][k] + f[k + 1][r] + s[r + 1] - s[l]);
    }
    return f[0][n - 1];
}
```

# 高精度

## 大整数：加、减、乘、除

输入为十进制非负整数字符串；减法允许负结果，除数必须大于 0，返回商和余数。Java 使用 BigInteger，Python 使用整数；除法不转浮点数。

```java
static String bigAdd(String a, String b) { return new BigInteger(a).add(new BigInteger(b)).toString(); }
static String bigSubtract(String a, String b) { return new BigInteger(a).subtract(new BigInteger(b)).toString(); }
static String bigMultiply(String a, String b) { return new BigInteger(a).multiply(new BigInteger(b)).toString(); }
static String[] bigDivide(String a, String b) {
    BigInteger divisor = new BigInteger(b); if (divisor.signum() <= 0) throw new IllegalArgumentException();
    BigInteger[] qr = new BigInteger(a).divideAndRemainder(divisor);
    return new String[]{qr[0].toString(), qr[1].toString()};
}
```

## 手写高精度：十进制数位数组

与 C++ 原模板的思路对应。数位低位在前，至少一位，输入须规范化（除零本身外无高位零）。实现加法、非负减法、乘非负小整数、除正小整数；减法要求 A≥B。Java 小整数 ≤10^9，中间进位用 long。

```java
static List<Integer> digits(String s) {
    List<Integer> a = new ArrayList<>(); for (int i = s.length() - 1; i >= 0; i--) a.add(s.charAt(i) - '0');
    trimDigits(a); return a;
}
static void trimDigits(List<Integer> a) { while (a.size() > 1 && a.get(a.size() - 1) == 0) a.remove(a.size() - 1); }
static List<Integer> addDigits(List<Integer> a, List<Integer> b) {
    List<Integer> c = new ArrayList<>(); int carry = 0;
    for (int i = 0; i < Math.max(a.size(), b.size()) || carry > 0; i++) {
        if (i < a.size()) carry += a.get(i); if (i < b.size()) carry += b.get(i);
        c.add(carry % 10); carry /= 10;
    }
    return c;
}
static List<Integer> subtractDigits(List<Integer> a, List<Integer> b) {
    List<Integer> c = new ArrayList<>(); int borrow = 0;
    for (int i = 0; i < a.size(); i++) {
        int x = a.get(i) - borrow - (i < b.size() ? b.get(i) : 0);
        c.add((x + 10) % 10); borrow = x < 0 ? 1 : 0;
    }
    trimDigits(c); return c;
}
static List<Integer> multiplyDigits(List<Integer> a, int b) {
    List<Integer> c = new ArrayList<>(); long carry = 0;
    for (int i = 0; i < a.size() || carry > 0; i++) {
        if (i < a.size()) carry += (long)a.get(i) * b;
        c.add((int)(carry % 10)); carry /= 10;
    }
    trimDigits(c); return c;
}
static List<Integer> divideDigits(List<Integer> a, int b, long[] remainder) {
    List<Integer> c = new ArrayList<>(); long r = 0;
    for (int i = a.size() - 1; i >= 0; i--) { r = r * 10 + a.get(i); c.add((int)(r / b)); r %= b; }
    Collections.reverse(c); trimDigits(c); remainder[0] = r; return c;
}
```

# 莫队

## 普通莫队：区间不同元素数

零基闭区间 [l,r]，支持任意整数值；返回原查询顺序的答案。左右端点移动时维护频率，右端点采用奇偶块排序。典型 n 与查询数同阶时 O((n+q)√n)。

```java
static int[] moDistinct(int[] a, int[][] queries) {
    int block = Math.max(1, (int)Math.sqrt(a.length));
    Integer[] order = new Integer[queries.length]; for (int i = 0; i < order.length; i++) order[i] = i;
    Arrays.sort(order, (i, j) -> {
        int x = queries[i][0] / block, y = queries[j][0] / block;
        if (x != y) return Integer.compare(x, y);
        return (x & 1) == 0 ? Integer.compare(queries[i][1], queries[j][1]) : Integer.compare(queries[j][1], queries[i][1]);
    });
    Map<Integer, Integer> freq = new HashMap<>(); int l = 0, r = -1; int[] ans = new int[queries.length];
    for (int id : order) {
        int ql = queries[id][0], qr = queries[id][1];
        while (l > ql) moAdjust(freq, a[--l], 1);
        while (r < qr) moAdjust(freq, a[++r], 1);
        while (l < ql) moAdjust(freq, a[l++], -1);
        while (r > qr) moAdjust(freq, a[r--], -1);
        ans[id] = freq.size();
    }
    return ans;
}
static void moAdjust(Map<Integer, Integer> freq, int value, int delta) {
    int count = freq.getOrDefault(value, 0) + delta;
    if (count == 0) freq.remove(value); else freq.put(value, count);
}
```

## 带修莫队：区间不同元素数

操作为 {0,l,r}=查询、{1,index,value}=单点赋值；零基闭区间，返回查询出现顺序的答案。时间指针通过交换旧值和新值实现前进、回滚。n、操作数同阶时典型 O(n^(5/3))，块长取 n^(2/3)；Java 依赖上一节 moAdjust。

```java
static int[] moDistinctWithUpdates(int[] initial, int[][] operations) {
    int[] a = initial.clone(); int block = Math.max(1, (int)Math.pow(Math.max(1, a.length), 2.0 / 3));
    List<int[]> changes = new ArrayList<>(), queries = new ArrayList<>();
    for (int[] op : operations) {
        if (op[0] == 0) queries.add(new int[]{op[1], op[2], changes.size(), queries.size()});
        else changes.add(new int[]{op[1], op[2]});
    }
    queries.sort((x, y) -> {
        int c = Integer.compare(x[0] / block, y[0] / block); if (c != 0) return c;
        c = Integer.compare(x[1] / block, y[1] / block); return c != 0 ? c : Integer.compare(x[2], y[2]);
    });
    Map<Integer, Integer> freq = new HashMap<>(); int l = 0, r = -1, time = 0; int[] ans = new int[queries.size()];
    for (int[] q : queries) {
        while (time < q[2]) moChange(a, changes.get(time++), l, r, freq);
        while (time > q[2]) moChange(a, changes.get(--time), l, r, freq);
        while (l > q[0]) moAdjust(freq, a[--l], 1);
        while (r < q[1]) moAdjust(freq, a[++r], 1);
        while (l < q[0]) moAdjust(freq, a[l++], -1);
        while (r > q[1]) moAdjust(freq, a[r--], -1);
        ans[q[3]] = freq.size();
    }
    return ans;
}
static void moChange(int[] a, int[] change, int l, int r, Map<Integer, Integer> freq) {
    int p = change[0];
    if (l <= p && p <= r) { moAdjust(freq, a[p], -1); moAdjust(freq, change[1], 1); }
    int old = a[p]; a[p] = change[1]; change[1] = old;
}
```

