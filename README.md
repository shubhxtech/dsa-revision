# 🧠 Last-Minute DSA + STL Revision

> **Who this is for:** Someone who has studied DSA before and needs fast recall — not first-time learners.
> Each section leads with a *mental model / visual intuition* first, then code, then traps.

---

## 📖 Table of Contents

| # | Section |
|---|---------|
| 1 | [⚡ Recognition Triggers](#1--recognition-triggers) |
| 2 | [🏗️ Data Structures & Techniques](#2--data-structures--techniques) |
| 3 | [🗺️ Graph Algorithms](#3--graph-algorithms) |
| 4 | [🧩 Dynamic Programming Patterns](#4--dynamic-programming-patterns) |
| 5 | [🔢 Math Tricks](#5--math-tricks) |
| 6 | [🔤 String Algorithms](#6--string-algorithms) |
| 7 | [🛠️ STL Cheat Sheet](#7--stl-cheat-sheet) |
| 8 | [⚖️ Confused-Technique Comparison Tables](#8--confused-technique-comparison-tables) |
| 9 | [💣 Common OA Traps](#9--common-oa-traps) |
| 10 | [🏁 Final 15–30 Minute Skim](#10--final-1530-minute-skim) |

---

## 1. ⚡ Recognition Triggers

> **Read this first. This is your mental lookup table.**

| Problem smells like... | Think |
|---|---|
| "Number of subarrays/substrings with sum/property X" | Prefix sum + hashmap, or two pointers if monotonic |
| "Range sum/update queries, many of them" | Fenwick / Segment Tree |
| "Range min/max, no updates" | Sparse Table (O(1) query) |
| "Range min/max WITH updates" | Segment Tree |
| "Next greater/smaller element" | Monotonic stack |
| "Sliding window max/min" | Monotonic deque |
| "Connectivity / grouping / dynamic components" | DSU |
| "Minimize max, or 'find smallest X such that condition holds'" | Binary search on answer |
| "Shortest path, all weights positive" | Dijkstra |
| "Shortest path, negative weights or need cycle detection" | Bellman-Ford |
| "Shortest path, all pairs, small N (≤400)" | Floyd-Warshall |
| "Shortest path, weights only 0/1" | 0-1 BFS (deque) |
| "Minimum cost to connect everything" | MST (Kruskal/Prim) |
| "Order tasks with dependencies" | Topological sort |
| "Subsequence + small N (≤20)" | Bitmask DP |
| "Count numbers in range with digit property" | Digit DP |
| "LIS / LDS or 'longest chain'" | Patience sorting O(n log n) |
| "Substring/pattern matching" | KMP / Z-function / hashing |
| "Palindromic substrings" | Manacher / expand-around-center / hashing |
| "XOR of subarray/pair maximization" | Trie on bits |
| "kth smallest / order statistics dynamically" | ordered_set (PBDS) or Fenwick over ranks |
| "Median of stream" | Two heaps |
| "Repeated identical subtree/array/state queries" | Memoize with map, or hashing states |
| "Tree + subtree sum/count queries" | Euler tour + Fenwick |
| "Tree + ancestor queries" | Binary lifting (LCA) |
| "Interval scheduling / merging / max non-overlapping" | Greedy sort by end time |
| "Something about parentheses/brackets validity" | Stack |
| "Matrix exponentiation smell (linear recurrence, huge n)" | Fast matrix power |
| "Modular arithmetic + large exponent" | Fast power + Fermat inverse |

---

## 2. 🏗️ Data Structures & Techniques

---

### 2.1 DSU (Union-Find / Disjoint Set)

#### 🧠 Mental Model

Imagine a forest of trees. Each tree = one group/component. Every node points to its root (the "representative").

```
Initially:    0  1  2  3  4      (each is its own group)
After unite(0,1):
              0←1  2  3  4      (1's parent is 0; 0 is the root)
After unite(2,3):
              0←1  2←3  4
After unite(1,2):
              0←1←2←3  4        (all in one component now)

Path compression: on find(3), we flatten:
              all point directly to 0
```

- `find(x)` → traces up to the root, compresses the path
- `unite(a, b)` → joins two trees; attach smaller tree under larger (union by rank/size)

**When to use:** Connectivity, Kruskal's MST, cycle detection in undirected graphs, "number of islands" with merges, offline connectivity.

**Complexity:** ~O(α(n)) ≈ O(1) amortized per op with both optimizations.

```cpp
struct DSU {
    vector<int> par, rnk;
    DSU(int n) : par(n), rnk(n, 0) { iota(par.begin(), par.end(), 0); }
    int find(int x) { return par[x] == x ? x : par[x] = find(par[x]); }  // path compression
    bool unite(int a, int b) {
        a = find(a); b = find(b);
        if (a == b) return false;           // already same component
        if (rnk[a] < rnk[b]) swap(a, b);   // attach smaller under larger
        par[b] = a;
        if (rnk[a] == rnk[b]) rnk[a]++;
        return true;
    }
    bool same(int a, int b) { return find(a) == find(b); }
};
```

> **⚠️ Traps:**
> - Always compress path in `find` — skipping degrades to O(n) per op.
> - If you need "size of component", maintain a `sz[]` updated in `unite`, not `rnk`.
> - DSU only ever merges — no un-union. For offline deletions, process queries in reverse.

**🔗 Practice Problems:**
- [Number of Provinces](https://leetcode.com/problems/number-of-provinces/) (LC 547) — Classic DSU
- [Redundant Connection](https://leetcode.com/problems/redundant-connection/) (LC 684) — Cycle detection
- [Number of Islands II](https://leetcode.com/problems/number-of-islands-ii/) (LC 305) — Online connectivity

---

### 2.2 Segment Tree

#### 🧠 Mental Model

Think of it as a **divide-and-conquer structure stored as an array**. Recursively split the array in half; each node stores the aggregate (sum/min/max/gcd) of its range. Node `i` has children `2i` (left) and `2i+1` (right).

```mermaid
graph TD
    A["[0..5] = 21"]
    A --> B["[0..2] = 7"]
    A --> C["[3..5] = 14"]
    B --> D["[0..1] = 3"]
    B --> E["[2] = 4"]
    D --> F["[0] = 2"]
    D --> G["[1] = 1"]
    C --> H["[3..4] = 9"]
    C --> I["[5] = 5"]
    H --> J["[3] = 3"]
    H --> K["[4] = 6"]
```

> **Query sum(1..4):** Walk the tree — collect nodes fully inside `[1..4]`, skip nodes fully outside.
> Nodes covered: `[1]=1`, `[2]=4`, `[3..4]=9` → **sum = 14** ✓

**Complexity:** Build O(n), query/update O(log n), space O(4n).

```cpp
struct SegTree {
    int n;
    vector<long long> t;
    SegTree(int n) : n(n), t(4 * n, 0) {}

    void update(int node, int l, int r, int pos, long long val) {
        if (l == r) { t[node] = val; return; }
        int mid = (l + r) / 2;
        if (pos <= mid) update(2*node, l, mid, pos, val);
        else            update(2*node+1, mid+1, r, pos, val);
        t[node] = t[2*node] + t[2*node+1];  // pull up
    }

    long long query(int node, int l, int r, int ql, int qr) {
        if (qr < l || r < ql) return 0;         // out of range — identity (0 for sum)
        if (ql <= l && r <= qr) return t[node]; // fully inside — return directly
        int mid = (l + r) / 2;
        return query(2*node, l, mid, ql, qr) + query(2*node+1, mid+1, r, ql, qr);
    }
    // Call as: update(1, 0, n-1, pos, val) / query(1, 0, n-1, ql, qr)
};
```

#### Lazy Propagation — Range update, Range query

**Concept:** Instead of updating every node in a range (O(n)), "defer" the update. Store pending updates in `lazy[]`. Propagate downward only when recursing into children ("push down").

```cpp
struct LazySegTree {
    int n;
    vector<long long> t, lz;
    LazySegTree(int n) : n(n), t(4*n, 0), lz(4*n, 0) {}

    void push(int node, int l, int r) {
        if (lz[node] == 0) return;
        t[node] += lz[node] * (r - l + 1);
        if (l != r) {
            lz[2*node]   += lz[node];
            lz[2*node+1] += lz[node];
        }
        lz[node] = 0;
    }

    void update(int node, int l, int r, int ql, int qr, long long val) {
        push(node, l, r);
        if (qr < l || r < ql) return;
        if (ql <= l && r <= qr) { lz[node] += val; push(node, l, r); return; }
        int mid = (l + r) / 2;
        update(2*node, l, mid, ql, qr, val);
        update(2*node+1, mid+1, r, ql, qr, val);
        t[node] = t[2*node] + t[2*node+1];
    }

    long long query(int node, int l, int r, int ql, int qr) {
        push(node, l, r);
        if (qr < l || r < ql) return 0;
        if (ql <= l && r <= qr) return t[node];
        int mid = (l + r) / 2;
        return query(2*node, l, mid, ql, qr) + query(2*node+1, mid+1, r, ql, qr);
    }
};
```

> **⚠️ Traps:**
> - `4*n` is the safe upper bound — don't under-allocate.
> - **Always call `push` before recursing** into a node's children.
> - For min/max trees, out-of-range identity = `+INF` / `-INF`, NOT `0`.
> - Pick 0-indexed or 1-indexed and stay consistent everywhere.

**🔗 Practice Problems:**
- [Range Sum Query — Mutable](https://leetcode.com/problems/range-sum-query-mutable/) (LC 307)
- [My Calendar III](https://leetcode.com/problems/my-calendar-iii/) (LC 732) — Lazy segtree
- [Count of Smaller Numbers After Self](https://leetcode.com/problems/count-of-smaller-numbers-after-self/) (LC 315)

---

### 2.3 Fenwick Tree / Binary Indexed Tree (BIT)

#### 🧠 Mental Model

Each index `i` is responsible for the sum of a range determined by its **lowest set bit** (`i & -i`).

```
Index:   1   2   3   4   5   6   7   8
Covers: [1] [1,2] [3] [1,4] [5] [5,6] [7] [1,8]

update(5, +3): 5 → 6 → 8   (each step: i += i & -i)
query(6):      6 → 4 → 0   (each step: i -= i & -i)  → sum of [1..6]
```

**Simpler and faster** than Segment Tree for prefix sum queries, but cannot do min/max.

```cpp
struct Fenwick {
    int n;
    vector<long long> bit;
    Fenwick(int n) : n(n), bit(n + 1, 0) {}

    void update(int i, long long delta) {  // 1-indexed
        for (; i <= n; i += i & (-i)) bit[i] += delta;
    }

    long long query(int i) {  // prefix sum [1..i]
        long long s = 0;
        for (; i > 0; i -= i & (-i)) s += bit[i];
        return s;
    }

    long long rangeQuery(int l, int r) { return query(r) - query(l - 1); }
};
```

**Range update + point query:** `update(l, +val); update(r+1, -val);` then point value at `i` = `query(i)`.

> **⚠️ Traps:**
> - Fenwick is **1-indexed** — index 0 causes infinite loop.
> - Cannot do range min/max — use Segment Tree instead.

**🔗 Practice Problems:**
- [Range Sum Query — Mutable](https://leetcode.com/problems/range-sum-query-mutable/) (LC 307)
- [Count of Smaller Numbers After Self](https://leetcode.com/problems/count-of-smaller-numbers-after-self/) (LC 315)

---

### 2.4 Sparse Table (Static RMQ)

#### 🧠 Mental Model

Precompute answers for all "power-of-2 length" ranges. For any query [l, r], two overlapping power-of-2 blocks cover it — and for idempotent functions (min, max, gcd), overlap doesn't matter since `min(x, x) = x`.

```
Array:  [3, 1, 4, 1, 5, 9, 2, 6]

st[0]:  [3, 1, 4, 1, 5, 9, 2, 6]   length 1
st[1]:  [1, 1, 1, 1, 5, 2, 2, _]   length 2
st[2]:  [1, 1, 1, 1, 2, 2, _, _]   length 4
st[3]:  [1, 1, _, _, _, _, _, _]   length 8

Query min(2..6): length=5, k=floor(log2(5))=2
  min(st[2][2], st[2][6-4+1]) = min(1, 1) = 1
```

```cpp
struct SparseTable {
    vector<vector<int>> st;
    vector<int> lg;

    SparseTable(vector<int>& a) {
        int n = a.size();
        lg.assign(n + 1, 0);
        for (int i = 2; i <= n; i++) lg[i] = lg[i/2] + 1;
        int K = lg[n] + 1;
        st.assign(K, vector<int>(n));
        st[0] = a;
        for (int k = 1; k < K; k++)
            for (int i = 0; i + (1 << k) <= n; i++)
                st[k][i] = min(st[k-1][i], st[k-1][i + (1 << (k-1))]);
    }

    int query(int l, int r) {  // inclusive, 0-indexed
        int k = lg[r - l + 1];
        return min(st[k][l], st[k][r - (1 << k) + 1]);
    }
};
```

> **⚠️ Trap:** Only for **idempotent** ops (min/max/gcd). Do NOT use for range sum — overlapping blocks double-count.

---

### 2.5 Trie (incl. XOR Trie)

#### 🧠 Mental Model

A tree where each path from root spells a word or number (in binary).

```
XOR Trie: Insert numbers bit by bit (MSB first).
5 = 101 → root → 1 → 0 → 1
3 = 011 → root → 0 → 1 → 1

To find max XOR with 5=101:
  At each level, greedily go to the OPPOSITE bit if it exists.
  bit 2: want 0 → go left if exists
  bit 1: want 1 → go right if exists
  etc.
```

```cpp
struct TrieNode { TrieNode* child[2] = {nullptr, nullptr}; };

struct XorTrie {
    TrieNode* root = new TrieNode();

    void insert(int num) {
        TrieNode* cur = root;
        for (int i = 31; i >= 0; i--) {
            int b = (num >> i) & 1;
            if (!cur->child[b]) cur->child[b] = new TrieNode();
            cur = cur->child[b];
        }
    }

    int maxXor(int num) {
        TrieNode* cur = root; int res = 0;
        for (int i = 31; i >= 0; i--) {
            int b = (num >> i) & 1;
            if (cur->child[1 - b]) { res |= (1 << i); cur = cur->child[1 - b]; }
            else cur = cur->child[b];
        }
        return res;
    }
};
```

**🔗 Practice Problems:**
- [Maximum XOR of Two Numbers in an Array](https://leetcode.com/problems/maximum-xor-of-two-numbers-in-an-array/) (LC 421)
- [Implement Trie (Prefix Tree)](https://leetcode.com/problems/implement-trie-prefix-tree/) (LC 208)
- [Word Search II](https://leetcode.com/problems/word-search-ii/) (LC 212)

---

### 2.6 Monotonic Stack / Deque

#### 🧠 Mental Model

A stack that **always stays sorted**. When a new element violates the order, pop the violating ones — each popped element has found its "next greater/smaller".

```
Next Greater Element for [2, 5, 1, 6, 3]:

i=0: push 2. stack: [2]
i=1: 5>2 → pop 2, NGE[2]=5. push 5. stack: [5]
i=2: 1<5 → push. stack: [5, 1]
i=3: 6>1 → pop 1, NGE[1]=6. 6>5 → pop 5, NGE[5]=6. push 6. stack: [6]
i=4: 3<6 → push. stack: [6, 3]

Remaining in stack → NGE = -1 (no greater to the right)
Result: NGE = [5, 6, 6, -1, -1]
```

```cpp
// Next Greater Element (to the right), O(n)
vector<int> nextGreater(vector<int>& a) {
    int n = a.size();
    vector<int> res(n, -1);
    stack<int> st;  // stores indices, values are DECREASING
    for (int i = 0; i < n; i++) {
        while (!st.empty() && a[st.top()] < a[i]) {
            res[st.top()] = a[i];
            st.pop();
        }
        st.push(i);
    }
    return res;
}

// Sliding window maximum using deque, O(n)
vector<int> slidingMax(vector<int>& a, int k) {
    deque<int> dq;  // stores indices; front = max of current window
    vector<int> res;
    for (int i = 0; i < (int)a.size(); i++) {
        while (!dq.empty() && dq.front() <= i - k) dq.pop_front();
        while (!dq.empty() && a[dq.back()] <= a[i]) dq.pop_back();
        dq.push_back(i);
        if (i >= k - 1) res.push_back(a[dq.front()]);
    }
    return res;
}
```

**🔗 Practice Problems:**
- [Daily Temperatures](https://leetcode.com/problems/daily-temperatures/) (LC 739)
- [Largest Rectangle in Histogram](https://leetcode.com/problems/largest-rectangle-in-histogram/) (LC 84)
- [Sliding Window Maximum](https://leetcode.com/problems/sliding-window-maximum/) (LC 239)

---

### 2.7 Two Pointers / Sliding Window

```cpp
int l = 0;
long long sum = 0;
int best = 0;
for (int r = 0; r < n; r++) {
    sum += a[r];
    while (sum > target) sum -= a[l++];  // shrink while invalid
    best = max(best, r - l + 1);
}
```

> **⚠️ Trap:** Only works when window validity is **monotonic** (adding can only worsen, removing can only improve). If non-monotonic → prefix sums + hashmap.

**🔗 Practice Problems:**
- [Minimum Size Subarray Sum](https://leetcode.com/problems/minimum-size-subarray-sum/) (LC 209)
- [Longest Substring Without Repeating Characters](https://leetcode.com/problems/longest-substring-without-repeating-characters/) (LC 3)
- [Fruit Into Baskets](https://leetcode.com/problems/fruit-into-baskets/) (LC 904)

---

### 2.8 Binary Search on Answer

**Key insight:** If you can CHECK whether answer `x` is feasible in O(f(n)), you can FIND the optimal in O(f(n) · log(range)).

```
"Can I do it in D days?" — binary search on D.
D: [NO, NO, NO, YES, YES, YES, YES]
                  ↑ answer
```

```cpp
int lo = LOW, hi = HIGH, ans = hi;
while (lo <= hi) {
    int mid = lo + (hi - lo) / 2;
    if (feasible(mid)) { ans = mid; hi = mid - 1; }
    else lo = mid + 1;
}
```

**🔗 Practice Problems:**
- [Koko Eating Bananas](https://leetcode.com/problems/koko-eating-bananas/) (LC 875)
- [Capacity to Ship Packages](https://leetcode.com/problems/capacity-to-ship-packages-within-d-days/) (LC 1011)
- [Split Array Largest Sum](https://leetcode.com/problems/split-array-largest-sum/) (LC 410)

---

## 3. 🗺️ Graph Algorithms

---

### 3.1 Dijkstra (non-negative weights)

#### 🧠 Mental Model

BFS with a **priority queue** (min-heap). Always process the node with the smallest known distance. Greedy works because edges are non-negative — you can't find a shorter path by going "backward".

```
Graph: 0 --(1)--> 1 --(2)--> 3
       0 --(4)--> 2 --(1)--> 3

From 0:
  Init: dist=[0,INF,INF,INF]. PQ: {(0,0)}
  Pop (0,0): relax 1→dist[1]=1, 2→dist[2]=4. PQ: {(1,1),(4,2)}
  Pop (1,1): relax 3→dist[3]=3. PQ: {(3,3),(4,2)}
  Pop (3,3): 3 already finalized. PQ: {(4,2)}
  Pop (4,2): 4+1=5 > dist[3]=3, skip.
  Final dist: [0, 1, 4, 3]
```

```cpp
vector<long long> dijkstra(int src, int n, vector<vector<pair<int,int>>>& adj) {
    vector<long long> dist(n, LLONG_MAX);
    priority_queue<pair<long long,int>, vector<pair<long long,int>>, greater<>> pq;
    dist[src] = 0;
    pq.push({0, src});
    while (!pq.empty()) {
        auto [d, u] = pq.top(); pq.pop();
        if (d > dist[u]) continue;  // stale entry
        for (auto [v, w] : adj[u]) {
            if (dist[u] + w < dist[v]) {
                dist[v] = dist[u] + w;
                pq.push({dist[v], v});
            }
        }
    }
    return dist;
}
```

> **⚠️ Trap:** Default PQ is max-heap — use `greater<>` for min-heap. The stale-entry check is essential.

**🔗 Practice Problems:**
- [Network Delay Time](https://leetcode.com/problems/network-delay-time/) (LC 743)
- [Path With Minimum Effort](https://leetcode.com/problems/path-with-minimum-effort/) (LC 1631)
- [Cheapest Flights Within K Stops](https://leetcode.com/problems/cheapest-flights-within-k-stops/) (LC 787)

---

### 3.2 Bellman-Ford

**Trigger:** Negative edge weights, or need to detect a **negative cycle**.

```cpp
bool bellmanFord(int n, int src, vector<array<int,3>>& edges, vector<long long>& dist) {
    dist.assign(n, LLONG_MAX); dist[src] = 0;
    for (int i = 0; i < n - 1; i++)
        for (auto& [u, v, w] : edges)
            if (dist[u] != LLONG_MAX && dist[u] + w < dist[v])
                dist[v] = dist[u] + w;
    // n-th pass: if we can still relax, there's a negative cycle
    for (auto& [u, v, w] : edges)
        if (dist[u] != LLONG_MAX && dist[u] + w < dist[v]) return false;
    return true;
}
```

**Complexity:** O(V·E).

---

### 3.3 Floyd-Warshall

**Trigger:** All-pairs shortest path, N ≤ ~400–500.

```cpp
// Initialize: dist[i][i]=0, dist[i][j]=edge weight or INF
for (int k = 0; k < n; k++)        // k MUST be outermost
    for (int i = 0; i < n; i++)
        for (int j = 0; j < n; j++)
            if (dist[i][k] < INF && dist[k][j] < INF)
                dist[i][j] = min(dist[i][j], dist[i][k] + dist[k][j]);
```

> **⚠️ Trap:** `k` **must be the outermost loop** — swapping loop order silently gives wrong answers.

---

### 3.4 0-1 BFS

**Trigger:** Edge weights only 0 or 1. O(V+E) instead of O((V+E) log V).

```
0-weight edge → push to FRONT (free move, high priority)
1-weight edge → push to BACK (normal move)
```

```cpp
deque<int> dq;
vector<int> dist(n, INT_MAX);
dist[src] = 0; dq.push_back(src);
while (!dq.empty()) {
    int u = dq.front(); dq.pop_front();
    for (auto [v, w] : adj[u]) {
        if (dist[u] + w < dist[v]) {
            dist[v] = dist[u] + w;
            if (w == 0) dq.push_front(v);
            else        dq.push_back(v);
        }
    }
}
```

**🔗 Practice Problems:**
- [Minimum Cost to Make at Least One Valid Path in a Grid](https://leetcode.com/problems/minimum-cost-to-make-at-least-one-valid-path-in-a-grid/) (LC 1368)

---

### 3.5 MST — Kruskal vs Prim

#### 🧠 Mental Model

**Minimum Spanning Tree** = cheapest set of edges connecting all nodes without cycles. Think: wiring a city's power grid with the least wire.

```
Kruskal: sort edges by weight, greedily add if no cycle (DSU check).
  Edges sorted: (1,A-B), (2,B-D), (3,C-D), (4,A-C)
  Add A-B: {A,B} {C} {D}
  Add B-D: {A,B,D} {C}
  Add C-D: {A,B,C,D} — done!
  Skip A-C: would form a cycle.
```

```cpp
// Kruskal (sort edges + DSU)
sort(edges.begin(), edges.end());  // {weight, u, v}
DSU dsu(n);
long long cost = 0;
for (auto& [w, u, v] : edges)
    if (dsu.unite(u, v)) cost += w;
// Complexity: O(E log E)

// Prim (min-heap, better for dense graphs)
vector<bool> inMST(n, false);
priority_queue<pair<int,int>, vector<pair<int,int>>, greater<>> pq;
pq.push({0, 0});
long long cost = 0;
while (!pq.empty()) {
    auto [w, u] = pq.top(); pq.pop();
    if (inMST[u]) continue;
    inMST[u] = true; cost += w;
    for (auto [v, wt] : adj[u])
        if (!inMST[v]) pq.push({wt, v});
}
// Complexity: O(E log V)
```

| | Kruskal | Prim |
|---|---|---|
| Better for | Sparse graphs | Dense graphs |
| Needs | Sorted edge list + DSU | Adjacency list + min-heap |
| Complexity | O(E log E) | O(E log V) |

**🔗 Practice Problems:**
- [Min Cost to Connect All Points](https://leetcode.com/problems/min-cost-to-connect-all-points/) (LC 1584)
- [Connecting Cities With Minimum Cost](https://leetcode.com/problems/connecting-cities-with-minimum-cost/) (LC 1135)

---

### 3.6 Topological Sort — Kahn's vs DFS

```
Kahn's (BFS-based):
  Track in-degrees. Push 0-indegree nodes into queue.
  Process node → reduce neighbors' in-degrees.
  If processed count < n → cycle exists!
```

```cpp
vector<int> indeg(n, 0);
for (auto& [u, v] : edges) indeg[v]++;
queue<int> q;
for (int i = 0; i < n; i++) if (indeg[i] == 0) q.push(i);
vector<int> order;
while (!q.empty()) {
    int u = q.front(); q.pop();
    order.push_back(u);
    for (int v : adj[u]) if (--indeg[v] == 0) q.push(v);
}
bool hasCycle = (order.size() != n);
```

**🔗 Practice Problems:**
- [Course Schedule](https://leetcode.com/problems/course-schedule/) (LC 207)
- [Course Schedule II](https://leetcode.com/problems/course-schedule-ii/) (LC 210)
- [Alien Dictionary](https://leetcode.com/problems/alien-dictionary/) (LC 269)

---

### 3.7 Cycle Detection

- **Undirected:** DSU (edge connecting two nodes in same component = cycle) OR DFS with parent-tracking.
- **Directed:** DFS with 3 colors — white/gray/black. Back-edge to a **gray** node = cycle.

---

### 3.8 LCA — Binary Lifting

#### 🧠 Mental Model

Precompute ancestors at power-of-2 jumps (like "jumping in halves" up the tree). To find LCA, equalize depths, then binary-jump both nodes up together until they meet.

```
Tree:    1
        / \
       2   3
      / \
     4   5

up[0][4]=2, up[0][2]=1, up[0][1]=1 (root points to itself)
up[1][4] = up[0][up[0][4]] = up[0][2] = 1

LCA(4, 3): depth[4]=2, depth[3]=1
  Lift 4 by 1: 4 → 2 (now both at depth 1)
  2 != 3, try k=LOG-1..0: up[0][2]=1, up[0][3]=1 → equal → LCA = up[0][2] = 1 ✓
```

```cpp
int LOG = 20;
vector<vector<int>> up(LOG, vector<int>(n, 0));
vector<int> depth(n, 0);
// Fill up[0][v] = parent[v] via DFS first
for (int k = 1; k < LOG; k++)
    for (int v = 0; v < n; v++)
        up[k][v] = up[k-1][up[k-1][v]];

int lca(int u, int v) {
    if (depth[u] < depth[v]) swap(u, v);
    int diff = depth[u] - depth[v];
    for (int k = 0; k < LOG; k++) if (diff & (1 << k)) u = up[k][u];
    if (u == v) return u;
    for (int k = LOG - 1; k >= 0; k--)
        if (up[k][u] != up[k][v]) { u = up[k][u]; v = up[k][v]; }
    return up[0][u];
}
```

**🔗 Practice Problems:**
- [Lowest Common Ancestor of a Binary Tree](https://leetcode.com/problems/lowest-common-ancestor-of-a-binary-tree/) (LC 236)
- [Kth Ancestor of a Tree Node](https://leetcode.com/problems/kth-ancestor-of-a-tree-node/) (LC 1483)

---

### 3.9 Euler Tour (Flatten Tree → Array)

**Trigger:** Subtree sum/count queries → convert to range queries on `[tin[v], tout[v]]`.

```
Tree: 1 → {2, 3},  2 → {4, 5}

DFS in: 1(0), 2(1), 4(2), 5(3), 3(4)
tin[2]=1, tout[2]=3 → subtree of 2 = positions [1..3] = {2, 4, 5} ✓
```

```cpp
int timer = 0;
vector<int> tin(n), tout(n);
void dfs(int u, int p) {
    tin[u] = timer++;
    for (int v : adj[u]) if (v != p) dfs(v, u);
    tout[u] = timer - 1;
}
// v is ancestor of u ⟺ tin[v] <= tin[u] && tout[u] <= tout[v]
```

---

### 3.10 SCC — Kosaraju's (brief)

Two DFS passes: (1) DFS original, push to stack on finish; (2) DFS transposed graph in stack-pop order — each DFS tree = one SCC. O(V+E). Condenses graph into a DAG.

---

## 4. 🧩 Dynamic Programming Patterns

---

### 4.1 Bitmask DP

#### 🧠 Mental Model

Use an integer's bits to represent a **subset of items**. `mask = 0b1010` means items 1 and 3 included (0-indexed). Add item `v` to mask: `mask | (1 << v)`. Check if item `v` in mask: `mask & (1 << v)`.

**Trigger:** N ≤ ~20, "visit all items" (TSP-style), subset partitioning.

```cpp
// TSP: dp[mask][u] = min cost to visit exactly cities in `mask`, ending at u
vector<vector<int>> dp(1 << n, vector<int>(n, INF));
dp[1][0] = 0;  // start at city 0
for (int mask = 1; mask < (1 << n); mask++)
    for (int u = 0; u < n; u++) {
        if (!(mask & (1 << u)) || dp[mask][u] == INF) continue;
        for (int v = 0; v < n; v++) {
            if (mask & (1 << v)) continue;
            int nmask = mask | (1 << v);
            dp[nmask][v] = min(dp[nmask][v], dp[mask][u] + cost[u][v]);
        }
    }
// Complexity: O(2^n * n^2)
```

**Submask enumeration** O(3^n total:
```cpp
for (int sub = mask; sub > 0; sub = (sub - 1) & mask) {
    // process subset sub of mask
}
```

**🔗 Practice Problems:**
- [Shortest Path Visiting All Nodes](https://leetcode.com/problems/shortest-path-visiting-all-nodes/) (LC 847)
- [Minimum Number of Work Sessions to Finish the Tasks](https://leetcode.com/problems/minimum-number-of-work-sessions-to-finish-the-tasks/) (LC 1986)

---

### 4.2 Digit DP

#### 🧠 Mental Model

Count numbers in [0, N] satisfying a digit property. Build the number digit by digit. Key flag: **`tight`** — are we still bounded by N?

```
Count numbers <= 325 with digit sum <= 7:
  At position 0, if we pick digit < 3, tight=false → digits 0..9 free afterward.
  If we pick digit = 3, tight=true → next digit bounded by 2.
  Memoize (pos, state) only when tight=false (tight=true states are unique to N's prefix).
```

```cpp
long long dp[20][2][MAXSTATE];
memset(dp, -1, sizeof dp);
function<long long(int,bool,int)> solve = [&](int pos, bool tight, int state) -> long long {
    if (pos == (int)digits.size()) return isValid(state);
    if (!tight && dp[pos][tight][state] != -1) return dp[pos][tight][state];
    int limit = tight ? digits[pos] : 9;
    long long res = 0;
    for (int d = 0; d <= limit; d++)
        res += solve(pos + 1, tight && (d == limit), transition(state, d));
    if (!tight) dp[pos][tight][state] = res;
    return res;
};
```

> **⚠️ Trap:** Do NOT memoize when `tight == true`.

**🔗 Practice Problems:**
- [Count Numbers with Unique Digits](https://leetcode.com/problems/count-numbers-with-unique-digits/) (LC 357)
- [Numbers At Most N Given Digit Set](https://leetcode.com/problems/numbers-at-most-n-given-digit-set/) (LC 902)

---

### 4.3 LIS in O(n log n) — Patience Sorting

#### 🧠 Mental Model

Place cards into piles: each card goes on the leftmost pile whose top card is >= current card. Number of piles = LIS length.

```
Sequence: [3, 1, 4, 1, 5, 9, 2, 6]
tails grows as:
  [3] → [1] → [1,4] → [1,1] → wait...
  tails[i] = smallest tail for LIS of length i+1

After all: tails.size() = LIS length
```

```cpp
vector<int> tails;
for (int x : a) {
    auto it = lower_bound(tails.begin(), tails.end(), x);  // strictly increasing
    if (it == tails.end()) tails.push_back(x);
    else *it = x;
}
int lisLength = tails.size();
// For non-decreasing LIS: use upper_bound instead of lower_bound
```

> **⚠️ Trap:** `lower_bound` = strictly increasing LIS. `upper_bound` = non-decreasing. This single swap is the most common LIS bug.

**🔗 Practice Problems:**
- [Longest Increasing Subsequence](https://leetcode.com/problems/longest-increasing-subsequence/) (LC 300)
- [Russian Doll Envelopes](https://leetcode.com/problems/russian-doll-envelopes/) (LC 354) — 2D LIS

---

### 4.4 Tree DP

```cpp
// Max weight independent set on tree
pair<long long,long long> dfs(int u, int p) {  // {take u, skip u}
    long long take = weight[u], skip = 0;
    for (int v : adj[u]) if (v != p) {
        auto [t, s] = dfs(v, u);
        take += s;           // if take u, children must be skipped
        skip += max(t, s);   // if skip u, children can be taken or skipped
    }
    return {take, skip};
}
```

**🔗 Practice Problems:**
- [House Robber III](https://leetcode.com/problems/house-robber-iii/) (LC 337)
- [Binary Tree Maximum Path Sum](https://leetcode.com/problems/binary-tree-maximum-path-sum/) (LC 124)

---

### 4.5 Knapsack Variants

| Variant | Inner loop direction | Why |
|---|---|---|
| 0/1 knapsack | Capacity **descending** | Prevents reusing same item |
| Unbounded knapsack | Capacity **ascending** | Allows reusing items |
| Bounded (count Ci) | Binary-split items, then 0/1 | Reduces to 0/1 knapsack |

**🔗 Practice Problems:**
- [Coin Change](https://leetcode.com/problems/coin-change/) (LC 322) — Unbounded
- [Partition Equal Subset Sum](https://leetcode.com/problems/partition-equal-subset-sum/) (LC 416) — 0/1

---

### 4.6 Interval DP

**Trigger:** Merge/partition a sequence into optimal cost — matrix chain, burst balloons, palindrome partitioning.

```cpp
for (int len = 2; len <= n; len++)
    for (int i = 0; i + len - 1 < n; i++) {
        int j = i + len - 1;
        for (int k = i; k < j; k++)
            dp[i][j] = min(dp[i][j], dp[i][k] + dp[k+1][j] + cost(i, k, j));
    }
// Complexity: O(n^3). Always iterate by INCREASING LENGTH.
```

**🔗 Practice Problems:**
- [Burst Balloons](https://leetcode.com/problems/burst-balloons/) (LC 312)
- [Strange Printer](https://leetcode.com/problems/strange-printer/) (LC 664)
- [Minimum Cost to Cut a Stick](https://leetcode.com/problems/minimum-cost-to-cut-a-stick/) (LC 1547)

---

## 5. 🔢 Math Tricks

---

### 5.1 Fast Power (Modular Exponentiation)

```
b^13 = b^(1101 in binary) = b^8 * b^4 * b^1
Just square and multiply: O(log e) multiplications.
```

```cpp
long long power(long long b, long long e, long long mod) {
    long long res = 1; b %= mod;
    while (e > 0) {
        if (e & 1) res = res * b % mod;
        b = b * b % mod;
        e >>= 1;
    }
    return res;
}
```

---

### 5.2 Modular Inverse

When `mod` is **prime**: `a^(-1) ≡ a^(mod-2) (mod p)` by Fermat's Little Theorem.

```cpp
long long modInverse(long long a, long long mod) { return power(a, mod - 2, mod); }
// Only works when mod is prime! Use extended Euclidean for non-prime mod.
```

---

### 5.3 nCr mod p

```cpp
const int MAXN = 200005; const long long MOD = 1e9 + 7;
vector<long long> fact(MAXN), inv_fact(MAXN);

void precompute() {
    fact[0] = 1;
    for (int i = 1; i < MAXN; i++) fact[i] = fact[i-1] * i % MOD;
    inv_fact[MAXN-1] = modInverse(fact[MAXN-1], MOD);
    for (int i = MAXN-2; i >= 0; i--) inv_fact[i] = inv_fact[i+1] * (i+1) % MOD;
}

long long nCr(int n, int r) {
    if (r < 0 || r > n) return 0;
    return fact[n] * inv_fact[r] % MOD * inv_fact[n-r] % MOD;
}
```

---

### 5.4 Sieve + Smallest Prime Factor

```cpp
vector<int> spf(MAXN);
void sieve() {
    iota(spf.begin(), spf.end(), 0);
    for (int i = 2; (long long)i * i < MAXN; i++)
        if (spf[i] == i)
            for (int j = i * i; j < MAXN; j += i)
                if (spf[j] == j) spf[j] = i;
}
vector<int> factorize(int n) {
    vector<int> f;
    while (n > 1) { f.push_back(spf[n]); n /= spf[n]; }
    return f;
}
```

---

### 5.5 GCD / LCM / Extended Euclid

```cpp
long long gcd(long long a, long long b) { return b ? gcd(b, a % b) : a; }
long long lcm(long long a, long long b) { return a / gcd(a, b) * b; }  // divide first!

long long extgcd(long long a, long long b, long long& x, long long& y) {
    if (b == 0) { x = 1; y = 0; return a; }
    long long x1, y1;
    long long g = extgcd(b, a % b, x1, y1);
    x = y1; y = x1 - (a / b) * y1;
    return g;
}
```

---

### 5.6 Bit Tricks

```cpp
__builtin_popcount(x)     // count set bits
__builtin_ctz(x)          // count trailing zeros
x & (-x)                  // isolates lowest set bit
x & (x - 1)               // removes lowest set bit
```

---

### 5.7 Matrix Exponentiation

**Trigger:** Linear recurrence with n up to 10^9 or 10^18. Represent as matrix M, compute M^n via fast power. O(k^3 log n) for k×k matrix. Used for Fibonacci in O(log n), counting paths in exactly k steps.

---

## 6. 🔤 String Algorithms

---

### 6.1 KMP (O(n+m))

```cpp
vector<int> computeLPS(string& pat) {
    int m = pat.size();
    vector<int> lps(m, 0);
    int len = 0, i = 1;
    while (i < m) {
        if (pat[i] == pat[len]) lps[i++] = ++len;
        else if (len) len = lps[len - 1];
        else lps[i++] = 0;
    }
    return lps;
}

vector<int> kmpSearch(string& text, string& pat) {
    vector<int> lps = computeLPS(pat), matches;
    int i = 0, j = 0;
    while (i < (int)text.size()) {
        if (text[i] == pat[j]) { i++; j++; }
        if (j == (int)pat.size()) { matches.push_back(i - j); j = lps[j - 1]; }
        else if (i < (int)text.size() && text[i] != pat[j]) {
            if (j) j = lps[j - 1]; else i++;
        }
    }
    return matches;
}
```

**🔗 Practice Problems:**
- [Find the Index of the First Occurrence in a String](https://leetcode.com/problems/find-the-index-of-the-first-occurrence-in-a-string/) (LC 28)
- [Repeated Substring Pattern](https://leetcode.com/problems/repeated-substring-pattern/) (LC 459)
- [Shortest Palindrome](https://leetcode.com/problems/shortest-palindrome/) (LC 214)

---

### 6.2 Z-Function

`z[i]` = length of longest substring starting at `i` matching a prefix of the string.

```cpp
vector<int> zFunction(string& s) {
    int n = s.size();
    vector<int> z(n, 0);
    int l = 0, r = 0;
    for (int i = 1; i < n; i++) {
        if (i < r) z[i] = min(r - i, z[i - l]);
        while (i + z[i] < n && s[z[i]] == s[i + z[i]]) z[i]++;
        if (i + z[i] > r) { l = i; r = i + z[i]; }
    }
    return z;
}
// Pattern search: run on (pattern + '#' + text), matches where z[i] == pattern.size()
```

---

### 6.3 Polynomial Hashing

```cpp
const long long MOD = 1e9 + 7, BASE = 131;
vector<long long> prefHash, powBase;

void buildHash(string& s) {
    int n = s.size();
    prefHash.assign(n + 1, 0); powBase.assign(n + 1, 1);
    for (int i = 0; i < n; i++) {
        prefHash[i+1] = (prefHash[i] * BASE + s[i]) % MOD;
        powBase[i+1] = powBase[i] * BASE % MOD;
    }
}

long long getHash(int l, int r) {
    long long h = (prefHash[r+1] - prefHash[l] * powBase[r-l+1]) % MOD;
    return (h + MOD) % MOD;
}
```

> **⚠️ Trap:** Use **double hashing** (two mod/base pairs) to avoid collisions.

---

### 6.4 Manacher

O(n) all palindromic substrings. Transform with `#` separators for even/odd uniformity. Only needed if n > ~5000 (use expand-around-center O(n²) by default — simpler).

---

## 7. 🛠️ STL Cheat Sheet

> 💡 **Philosophy:** STL is powerful but has sharp edges. This section explains *what* each function actually returns, *why* to use it, and the **non-trivial patterns** you'll actually need in contests and interviews.

---

### 7.1 Binary Search on Sorted Vectors

#### What do `lower_bound` and `upper_bound` actually return?

```
Vector v = [1, 3, 3, 5, 7]
            0  1  2  3  4   ← indices

lower_bound(v, 3) → iterator to index 1  (first element >= 3)
upper_bound(v, 3) → iterator to index 3  (first element >  3)
lower_bound(v, 4) → iterator to index 3  (no 4; points to next bigger: 5)
lower_bound(v, 8) → v.end()              (bigger than everything)
```

```cpp
// --- Basics ---
auto it = lower_bound(v.begin(), v.end(), x); // first element >= x
auto it = upper_bound(v.begin(), v.end(), x); // first element >  x

// Convert iterator to index:
int idx = lower_bound(v.begin(), v.end(), x) - v.begin();

// Check if x actually exists:
bool exists = binary_search(v.begin(), v.end(), x);
// Or equivalently:
bool exists = (it != v.end() && *it == x);

// Count occurrences of x in sorted v:
int cnt = upper_bound(v.begin(), v.end(), x) - lower_bound(v.begin(), v.end(), x);

// Find last element <= x:
auto it = upper_bound(v.begin(), v.end(), x);
if (it != v.begin()) --it;  // *it is now the largest element <= x
```

#### Searching on structs / pairs

```cpp
vector<pair<int,int>> v = {{1,10},{3,20},{3,30},{5,40}};

// Find first pair whose .first >= 3:
auto it = lower_bound(v.begin(), v.end(), make_pair(3, INT_MIN));
// INT_MIN ensures we get the very first pair with .first == 3

// Custom field search with comparator:
struct Item { int id, val; };
vector<Item> items;
auto it = lower_bound(items.begin(), items.end(), 5,
    [](const Item& a, int b) { return a.val < b; }); // first Item with .val >= 5
```

#### Binary search on sets and maps (use MEMBER, not `std::`)

```cpp
set<int> s = {1, 3, 5, 7};

// CORRECT — O(log n):
auto it = s.lower_bound(4);  // → iterator to 5
auto it = s.upper_bound(3);  // → iterator to 5

// WRONG — O(n), iterates linearly through set:
auto it = std::lower_bound(s.begin(), s.end(), 4);  // ← DO NOT USE on set/map
```

---

### 7.2 `sort` — Custom Comparators

#### The comparator contract: `cmp(a, b)` must return `true` if `a` should come **before** `b`.

```cpp
vector<int> v = {5, 1, 4, 2, 3};

// Sort ascending (default):
sort(v.begin(), v.end());

// Sort descending:
sort(v.begin(), v.end(), greater<int>());
// Or with lambda:
sort(v.begin(), v.end(), [](int a, int b){ return a > b; });

// Sort vector of pairs by second element, then by first descending:
vector<pair<int,int>> p;
sort(p.begin(), p.end(), [](const auto& a, const auto& b){
    if (a.second != b.second) return a.second < b.second;  // by .second asc
    return a.first > b.first;                               // tie-break: .first desc
});

// Sort by a custom struct field:
struct Task { int deadline, profit; };
vector<Task> tasks;
sort(tasks.begin(), tasks.end(),
    [](const Task& a, const Task& b){ return a.deadline < b.deadline; });

// Sort indices of array v by their values (without moving v):
vector<int> idx(n);
iota(idx.begin(), idx.end(), 0);
sort(idx.begin(), idx.end(), [&](int i, int j){ return v[i] < v[j]; });
// idx[0] is now the index of the smallest element in v

// stable_sort: preserves relative order of equal elements:
stable_sort(v.begin(), v.end(), cmp);
```

> **⚠️ Trap:** Comparators must be **strict weak ordering** — never return `true` when `a == b` (e.g., `return a <= b` is UB). Always use strict `<` or `>`.

---

### 7.3 `priority_queue` — All Forms

#### Intuition: PQ comparator is the **opposite** of sort comparator.
- In `sort`: `cmp(a,b)=true` means a comes first (a is "less")
- In PQ: `cmp(a,b)=true` means a has **lower priority** → b goes to top

```cpp
// Max-heap (default — largest on top):
priority_queue<int> pq;
pq.push(3); pq.push(1); pq.push(5);
pq.top();   // 5

// Min-heap (smallest on top):
priority_queue<int, vector<int>, greater<int>> pq;

// Min-heap of pairs {distance, node} — classic Dijkstra:
priority_queue<pair<int,int>, vector<pair<int,int>>, greater<>> pq;
pq.push({0, src});
auto [dist, node] = pq.top(); pq.pop();

// Custom comparator via LAMBDA (need decltype):
auto cmp = [](const pair<int,int>& a, const pair<int,int>& b) {
    return a.second > b.second;  // min-heap by .second
};
priority_queue<pair<int,int>, vector<pair<int,int>>, decltype(cmp)> pq(cmp);

// Custom comparator via STRUCT (cleaner for complex logic):
struct Cmp {
    bool operator()(const pair<int,int>& a, const pair<int,int>& b) const {
        if (a.second != b.second) return a.second > b.second; // min by .second
        return a.first > b.first;                             // tie: min by .first
    }
};
priority_queue<pair<int,int>, vector<pair<int,int>>, Cmp> pq;

// K largest elements using a MIN-heap of size K:
priority_queue<int, vector<int>, greater<int>> minPQ;
for (int x : arr) {
    minPQ.push(x);
    if ((int)minPQ.size() > k) minPQ.pop();  // evict smallest
}
// minPQ now contains the K largest elements; minPQ.top() = kth largest
```

---

### 7.4 `set` / `multiset` / `map` Essentials

```cpp
// ---- set ----
set<int> s;
s.insert(x);
s.erase(x);           // removes x if it exists, no-op otherwise
s.count(x);           // 0 or 1
s.find(x);            // iterator to x, or s.end() if not found

// Neighbour queries — very useful in interviews:
auto it = s.lower_bound(x);   // first element >= x  (member fn, O(log n))
auto it = s.upper_bound(x);   // first element >  x
// Largest element <= x:
auto it = s.upper_bound(x);
if (it != s.begin()) { --it; /* *it is largest <= x */ }
// Smallest element > x (next element after x):
auto it = s.upper_bound(x);  // directly points there

// ---- multiset ----
multiset<int> ms;
ms.insert(x);               // allows duplicates
ms.count(x);                // count of x
ms.erase(ms.find(x));       // ← erase EXACTLY ONE copy of x
ms.erase(x);                // ← erases ALL copies of x — COMMON BUG!

// ---- map ----
map<int,int> mp;
mp[x]++;                         // creates mp[x]=0 if absent, then ++
mp.count(x);                     // 1 if exists, 0 if not (no side effect)
if (mp.find(x) != mp.end()) {}   // safe existence check

// Iterate in sorted order:
for (auto& [key, val] : mp) { ... }  // ascending by key automatically

// Safe erase while iterating (classic pattern):
for (auto it = mp.begin(); it != mp.end(); ) {
    if (shouldRemove(*it)) it = mp.erase(it);  // erase returns next iterator
    else ++it;
}

// Get all values for a key range [lo, hi]:
for (auto it = mp.lower_bound(lo); it != mp.upper_bound(hi); ++it) {
    // process it->first, it->second
}
```

---

### 7.5 `unordered_map` / `unordered_set` — When and How

```cpp
// O(1) average lookup — use when you don't need sorted order
unordered_map<int,int> mp;
unordered_set<int> st;

// Avoid TLE from hash collisions (adversarial inputs):
mp.reserve(1 << 18);      // pre-allocate buckets (use power of 2)
mp.max_load_factor(0.25); // fewer collisions at cost of memory

// Custom hash for pair<int,int> (no built-in hash!):
struct PairHash {
    size_t operator()(const pair<int,int>& p) const {
        // Combine two hashes using XOR + shift trick:
        return hash<long long>()(((long long)p.first << 32) ^ (unsigned)p.second);
    }
};
unordered_map<pair<int,int>, int, PairHash> mp;

// Custom hash for any struct:
struct Node { int x, y, state; };
struct NodeHash {
    size_t operator()(const Node& n) const {
        size_t h = 0;
        h ^= hash<int>()(n.x)   + 0x9e3779b9 + (h<<6) + (h>>2);
        h ^= hash<int>()(n.y)   + 0x9e3779b9 + (h<<6) + (h>>2);
        h ^= hash<int>()(n.state) + 0x9e3779b9 + (h<<6) + (h>>2);
        return h;
    }
};
```

> **⚠️ Rule of thumb:** If TLE on `unordered_map` with large input → switch to `map` (log n but no hash attacks) or reserve + lower load factor.

---

### 7.6 Frequently-Forgotten STL (with explanations)

```cpp
// accumulate — sum / product / custom fold:
long long sum = accumulate(v.begin(), v.end(), 0LL);     // 0LL: avoids int overflow!
long long prod = accumulate(v.begin(), v.end(), 1LL, multiplies<long long>());
string joined = accumulate(words.begin(), words.end(), string(""),
    [](const string& a, const string& b){ return a + " " + b; });

// Deduplication (must sort first — unique only removes CONSECUTIVE duplicates):
sort(v.begin(), v.end());
v.erase(unique(v.begin(), v.end()), v.end());
// After this, v has unique elements in sorted order

// Coordinate compression (rank of each element):
vector<int> sorted_v = v;
sort(sorted_v.begin(), sorted_v.end());
sorted_v.erase(unique(sorted_v.begin(), sorted_v.end()), sorted_v.end());
auto rank = [&](int x) {
    return lower_bound(sorted_v.begin(), sorted_v.end(), x) - sorted_v.begin();
};

// next_permutation / prev_permutation:
vector<int> perm = {1, 2, 3};
do {
    // process this permutation
} while (next_permutation(perm.begin(), perm.end()));
// Must start from sorted order to iterate ALL permutations

// max_element / min_element:
int maxVal = *max_element(v.begin(), v.end());
int minIdx = min_element(v.begin(), v.end()) - v.begin();  // index of min

// nth_element — partial sort, O(n) average (for kth smallest):
nth_element(v.begin(), v.begin() + k, v.end());
int kth = v[k];  // v[k] is now the (k+1)-th smallest; elements around it are unordered

// fill / iota:
fill(v.begin(), v.end(), 0);         // set all to 0
iota(v.begin(), v.end(), 1);         // fill with 1, 2, 3, ...

// rotate:
rotate(v.begin(), v.begin() + k, v.end());  // left-rotate by k positions

// count_if:
int neg = count_if(v.begin(), v.end(), [](int x){ return x < 0; });

// memset (careful!):
memset(arr, 0, sizeof(arr));    // fills with 0 — safe
memset(arr, -1, sizeof(arr));   // fills with -1 (0xFF bytes) — safe for -1
memset(arr, 0x3f, sizeof(arr)); // fills each byte with 0x3f → arr[i] = 0x3f3f3f3f ≈ 1e9
// NEVER use memset to set arbitrary int values like 5 — it sets bytes, not ints!

// __int128 (no cin/cout — print manually):
__int128 big = (__int128)1e36;
auto print128 = [](auto x) {
    if (x < 0) { cout << '-'; x = -x; }
    if (x > 9) print128(x / 10);
    cout << (char)('0' + x % 10);
};

// bitset — fast for set operations over large fixed-size domains:
bitset<100001> sieve;
sieve.set();           // all 1s
sieve.reset(0); sieve.reset(1);
sieve.count();         // number of set bits
sieve[i];              // access bit i
(a & b).count();       // count elements in intersection of two bitsets
```

---

### 7.7 PBDS ordered_set (Policy-Based)

> An augmented Red-Black Tree that supports **order statistics** — find k-th element or rank of an element — both in O(log n).

```cpp
#include <ext/pb_ds/assoc_container.hpp>
#include <ext/pb_ds/tree_policy.hpp>
using namespace __gnu_pbds;
typedef tree<int, null_type, less<int>, rb_tree_tag,
             tree_order_statistics_node_update> ordered_set;

ordered_set os;
os.insert(3); os.insert(1); os.insert(5); os.insert(2);

os.order_of_key(3);     // → 2  (count of elements strictly < 3: {1, 2})
os.order_of_key(6);     // → 4  (all elements are < 6)
*os.find_by_order(0);   // → 1  (0-indexed: smallest)
*os.find_by_order(2);   // → 3  (2nd index: 3rd smallest)

// Supports set operations (lower_bound, upper_bound, etc.) like std::set
os.erase(3);
os.find(5);  // like set::find

// For DUPLICATES — use pair<int,int> with a unique second key:
ordered_set<pair<int,int>> os2;
int uid = 0;
os2.insert({val, uid++});  // unique second element prevents collisions
// Now order_of_key({val, 0}) = count of elements with value < val
```

> **⚠️ Trap:** Only works with GNU G++ (usually fine on Codeforces/CSES; verify for HackerRank/custom OA judges). Have a Fenwick-over-compressed-ranks fallback.

---

### 7.8 Lambda & Functional Patterns

```cpp
// Basic lambda:
auto square = [](int x) { return x * x; };

// Capture by reference [&] — can modify outer variables:
int total = 0;
for_each(v.begin(), v.end(), [&](int x){ total += x; });

// Capture by value [=] — read-only snapshot:
auto adder = [offset = 10](int x) { return x + offset; };  // capture with initializer

// Generic lambda (C++14) — works on any type:
auto printPair = [](const auto& p) { cout << p.first << ' ' << p.second << '\n'; };

// Recursive lambda (two ways):
// Way 1 — pass self explicitly:
auto dfs = [&](auto&& self, int u, int parent) -> void {
    visited[u] = true;
    for (int v : adj[u]) if (v != parent) self(self, v, u);
};
dfs(dfs, 0, -1);

// Way 2 — std::function (slower due to type erasure, but cleaner):
function<int(int)> fib = [&](int n) -> int {
    return n < 2 ? n : fib(n-1) + fib(n-2);
};

// Lambda as comparator in sort:
vector<string> words;
sort(words.begin(), words.end(),
    [](const string& a, const string& b){ return a.size() < b.size(); });

// Lambda in transform:
vector<int> squared(n);
transform(v.begin(), v.end(), squared.begin(), [](int x){ return x * x; });
```

---

## 8. ⚖️ Confused-Technique Comparison Tables

### Shortest Path

| Situation | Algorithm | Complexity |
|---|---|---|
| Non-negative weights, single source | Dijkstra | O((V+E) log V) |
| Negative weights / negative cycle detection | Bellman-Ford | O(V·E) |
| All-pairs, small V (≤ ~400–500) | Floyd-Warshall | O(V³) |
| Weights only 0/1 | 0-1 BFS | O(V+E) |
| Unweighted | Plain BFS | O(V+E) |
| DAG | Topological order + relax | O(V+E) |

---

### Range Query Structures

| Need | Structure |
|---|---|
| Static, range min/max/gcd, no updates | Sparse Table (O(1) query) |
| Point update + range sum | Fenwick Tree |
| Point update + range min/max | Segment Tree |
| Range update + range query | Segment Tree with lazy propagation |
| Range update + point query | Fenwick over difference array |

---

### MST

| Situation | Use |
|---|---|
| Sparse graph, edge list | Kruskal (sort + DSU) |
| Dense graph, adjacency list | Prim (heap-based) |

---

### Container Selection

| Need | Container |
|---|---|
| Sorted, unique, O(log n) ops | `set` |
| Fastest average lookup | `unordered_set` |
| Duplicates, sorted | `multiset` |
| Build once, query many times | `vector` + `sort` + `lower_bound` |
| k-th order stat dynamically | PBDS `ordered_set` |

---

### DP Style

| Situation | Prefer |
|---|---|
| Not all states reachable | Memoization |
| Need rolling array / space opt | Tabulation |
| Deep recursion risk | Tabulation |

---

### String Matching

| Situation | Prefer |
|---|---|
| Single pattern in text | KMP or Z-function |
| Compare many substrings | Hashing (O(1) compare) |
| Guaranteed no collision | KMP/Z (deterministic) |

---

## 9. 💣 Common OA Traps

> Things that cause WA/TLE even when core logic is correct.

- **Integer overflow:** `int` overflows at ~2.1×10⁹. Use `long long` for any product/sum of large numbers.
- **Negative modulo in C++:** `(a - b) % m` can be negative → `((a - b) % m + m) % m`.
- **Stack overflow:** DFS on chain/skewed tree with 10⁵ nodes → convert to iterative.
- **I/O speed:** `ios_base::sync_with_stdio(false); cin.tie(nullptr);` + `'\n'` not `endl`.
- **`multiset::erase(value)` deletes ALL** → use `erase(iterator)` for one.
- **`unique()` needs sort first** — only removes consecutive duplicates.
- **Floyd-Warshall: `k` must be outermost loop** — swapping silently gives wrong answers.
- **Fenwick is 1-indexed** — index 0 → infinite loop.
- **Segment tree identity:** min/max trees need `+INF`/`-INF` for out-of-range, not `0`.
- **Not resetting global state** across multiple test cases.
- **`adj(n)` with 1-indexed nodes** → size should be `n+1`.
- **Floating point `==`** → never; use `abs(a - b) < 1e-9`.
- **"Obvious" greedy without proof** → stress-test mentally on small examples.

---

## 10. 🏁 Final 15–30 Minute Skim

> **Read this section right before the OA starts.**

### Quick Recognition Table

| See in problem | Think immediately |
|---|---|
| Range queries + updates | Fenwick (sum) / Seg Tree (min/max/lazy) / Sparse Table (static) |
| Connectivity / grouping | DSU |
| Next greater/smaller, sliding window max | Monotonic stack/deque |
| "Minimize the max" / "smallest X satisfying P" | Binary search on answer |
| Shortest path (positive weights) | Dijkstra |
| Shortest path (negative weights) | Bellman-Ford |
| All-pairs, small N | Floyd-Warshall |
| 0/1 edge weights | 0-1 BFS |
| Connect all nodes cheaply | MST (Kruskal/Prim) |
| N ≤ 20, subsets | Bitmask DP |
| XOR max pair/subarray | Trie on bits |
| k-th order stat dynamically | PBDS ordered_set / Fenwick over ranks |

---

### Syntax to Not Fumble

```
lower_bound  →  first >= x     |     upper_bound  →  first > x
Set member:   s.lower_bound(x)     NOT std::lower_bound (O(n) on set!)
Min-heap:     priority_queue<int, vector<int>, greater<int>>
Multiset:     erase(iterator) = 1 copy,  erase(value) = ALL copies
LIS strictly increasing  → lower_bound
LIS non-decreasing       → upper_bound
Negative mod:  ((a % m) + m) % m
Fast I/O:      ios_base::sync_with_stdio(false); cin.tie(nullptr); + '\n' not endl
Fenwick        → 1-indexed
Floyd-Warshall → k loop is OUTERMOST
When in doubt → long long
```

---

### Pre-Submit Checklist

1. ✅ Check constraints → right algorithm complexity? (10⁸ ops/sec rule of thumb)
2. ✅ Off-by-one in loop bounds and binary search?
3. ✅ Overflow on multiplication/summation → `long long`?
4. ✅ Multiple test cases → global/static state reset?
5. ✅ Edge cases: n=1, empty input, all-equal, single node, negative numbers?

---

*Good luck! You've got this. 🚀*
