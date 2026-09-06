# Last-Minute DSA + STL Revision — InMobi OA

*For someone who already knows DSA cold. No theory, no basics, just fast recall triggers, templates, and traps. Read top to bottom once, then jump straight to "Final 15–30 Min" section right before the OA.*

---

## How this doc is organized
1. Recognition triggers ("if you see X in the problem, think Y")
2. Data structures & techniques — intuition + template + traps
3. Graph algorithm zoo + when to use which
4. DP patterns
5. Math tricks
6. STL cheat-sheet (the actual syntax you forget)
7. Confused-technique comparison tables
8. Common OA traps / WA-causes
9. Final 15–30 minute skim

---

## 1. Recognition Triggers (read this first, seriously)

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

## 2. Data Structures & Techniques

### 2.1 DSU (Union-Find)

**Intuition:** Maintain disjoint groups; answer "same group?" and "merge groups" near O(1) amortized.

**Trigger:** Dynamic connectivity, Kruskal's MST, cycle detection in undirected graph, "number of islands with merging", offline connectivity queries processed in some order.

**Complexity:** ~O(α(n)) per op (inverse Ackermann, effectively constant) with path compression + union by rank/size.

```cpp
struct DSU {
    vector<int> par, rnk;
    DSU(int n) : par(n), rnk(n, 0) { iota(par.begin(), par.end(), 0); }
    int find(int x) { return par[x] == x ? x : par[x] = find(par[x]); }
    bool unite(int a, int b) {
        a = find(a); b = find(b);
        if (a == b) return false;
        if (rnk[a] < rnk[b]) swap(a, b);
        par[b] = a;
        if (rnk[a] == rnk[b]) rnk[a]++;
        return true;
    }
};
```

**Traps:**
- Always compress path in `find` (recursive `par[x]=find(par[x])`), else it degrades to O(n).
- If you need "size of component", maintain a `sz[]` array updated in `unite`, not `rnk`.
- DSU only ever merges — no "un-union". If problem needs offline deletions, process queries in reverse, or use small-to-large / persistent DSU (rare in OAs).

---

### 2.2 Segment Tree

**Intuition:** Binary tree over array indices; each node stores an aggregate (sum/min/max/gcd) of its range. Update and query in O(log n).

**Trigger:** Range query + range/point update, need something more flexible than Fenwick (min/max, or lazy range updates).

**Complexity:** Build O(n), query/update O(log n), space O(4n).

```cpp
// Point update, range sum query
struct SegTree {
    int n; vector<long long> t;
    SegTree(int n) : n(n), t(4 * n, 0) {}
    void update(int node, int l, int r, int pos, long long val) {
        if (l == r) { t[node] = val; return; }
        int mid = (l + r) / 2;
        if (pos <= mid) update(2*node, l, mid, pos, val);
        else update(2*node+1, mid+1, r, pos, val);
        t[node] = t[2*node] + t[2*node+1];
    }
    long long query(int node, int l, int r, int ql, int qr) {
        if (qr < l || r < ql) return 0;      // identity for sum; use INF/-INF for min/max
        if (ql <= l && r <= qr) return t[node];
        int mid = (l + r) / 2;
        return query(2*node, l, mid, ql, qr) + query(2*node+1, mid+1, r, ql, qr);
    }
};
```

**Lazy propagation (range update, range query — the part people forget under pressure):**
```cpp
struct LazySegTree {
    int n; vector<long long> t, lz;
    LazySegTree(int n) : n(n), t(4*n, 0), lz(4*n, 0) {}
    void push(int node, int l, int r) {
        if (lz[node] == 0) return;
        t[node] += lz[node] * (r - l + 1);
        if (l != r) { lz[2*node] += lz[node]; lz[2*node+1] += lz[node]; }
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

**Traps:**
- `4*n` sizing is a safe upper bound; don't under-allocate.
- Always `push` before reading/recursing into a node's children.
- For min/max segment trees, identity element on out-of-range must be +INF/-INF, not 0.
- 0-indexed vs 1-indexed array — decide once, stay consistent through build/update/query calls.

---

### 2.3 Fenwick Tree / Binary Indexed Tree (BIT)

**Intuition:** Implicit tree using bit tricks; each index stores partial sum of a range determined by its lowest set bit. Simpler and faster than segment tree, but only works for invertible/prefix-composable operations (sum, XOR — NOT min/max directly).

**Trigger:** Point update + prefix sum query, counting inversions, "range update + point query" (via difference array trick).

**Complexity:** O(log n) per operation, O(n) space. Much smaller constant than segment tree.

```cpp
struct Fenwick {
    int n; vector<long long> bit;
    Fenwick(int n) : n(n), bit(n + 1, 0) {}
    void update(int i, long long delta) {          // 1-indexed
        for (; i <= n; i += i & (-i)) bit[i] += delta;
    }
    long long query(int i) {                        // prefix sum [1..i]
        long long s = 0;
        for (; i > 0; i -= i & (-i)) s += bit[i];
        return s;
    }
    long long rangeQuery(int l, int r) { return query(r) - query(l - 1); }
};
```

**Range update + point query variant:** maintain Fenwick over the *difference array*: `update(l, +val); update(r+1, -val);` then point value = `query(i)`.

**Range update + range query variant:** needs two Fenwick trees (standard trick — look up if truly needed, rare in OAs).

**Counting inversions with Fenwick:** coordinate-compress values, then for each element (left to right) query how many *greater* elements already inserted (`i - query(rank)`), then insert.

**Traps:**
- Fenwick is 1-indexed by convention — index 0 breaks the `i & (-i)` trick (infinite loop / no-op).
- Cannot do range-min/max Fenwick trivially (no inverse operation) — use Segment Tree or Sparse Table instead.

---

### 2.4 Sparse Table (Static RMQ)

**Intuition:** Precompute answers for all ranges of length 2^k. Any range [l,r] can be covered by 2 overlapping power-of-2 ranges for idempotent ops (min/max/gcd/AND/OR — overlap doesn't break correctness).

**Trigger:** Range min/max/gcd query, NO updates, need O(1) per query.

**Complexity:** Build O(n log n), query O(1), space O(n log n).

```cpp
struct SparseTable {
    vector<vector<int>> st; vector<int> lg;
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
    int query(int l, int r) { // inclusive, 0-indexed
        int k = lg[r - l + 1];
        return min(st[k][l], st[k][r - (1 << k) + 1]);
    }
};
```

**Trap:** Only valid for idempotent functions. Do NOT use for range **sum** (overlap double-counts) — use Fenwick/prefix sums instead.

---

### 2.5 Trie (incl. XOR Trie)

**Intuition:** Tree where each path from root spells a prefix. O(L) insert/search where L = string length or bit-length.

**Trigger:** Prefix search, autocomplete-style problems, **maximum XOR pair/subarray** (classic: insert numbers in binary, 32 bits from MSB, greedily go opposite bit).

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
**Trap:** Fix bit-width (e.g., 31 or 32) consistently; insert root sentinel before querying first element so `maxXor` never dereferences null.

---

### 2.6 Monotonic Stack / Deque

**Intuition:** Maintain elements in increasing/decreasing order; pop elements that can never be the answer for future queries. Each element pushed/popped once → O(n) total.

**Trigger:** Next greater/smaller element, largest rectangle in histogram, stock span, **sliding window max/min**.

```cpp
// Next Greater Element (to the right), O(n)
vector<int> nextGreater(vector<int>& a) {
    int n = a.size();
    vector<int> res(n, -1); stack<int> st;   // stores indices
    for (int i = 0; i < n; i++) {
        while (!st.empty() && a[st.top()] < a[i]) { res[st.top()] = a[i]; st.pop(); }
        st.push(i);
    }
    return res;
}

// Sliding window maximum, O(n)
vector<int> slidingMax(vector<int>& a, int k) {
    deque<int> dq; vector<int> res;   // stores indices, decreasing values
    for (int i = 0; i < a.size(); i++) {
        while (!dq.empty() && a[dq.back()] <= a[i]) dq.pop_back();
        dq.push_back(i);
        if (dq.front() <= i - k) dq.pop_front();
        if (i >= k - 1) res.push_back(a[dq.front()]);
    }
    return res;
}
```
**Trap:** For "next greater" go left→right with decreasing stack; for "previous greater" go right→left, or push/pop conditions flip. Draw it if unsure — don't trust memory of the direction blindly.

---

### 2.7 Two Pointers / Sliding Window

**Trigger:** Contiguous subarray with sum/count constraint, "smallest/largest window satisfying condition", pair-sum in sorted array.

**Pattern (variable-size window, monotonic condition):**
```cpp
int l = 0; long long sum = 0; int best = 0;
for (int r = 0; r < n; r++) {
    sum += a[r];
    while (sum > target) sum -= a[l++];   // shrink while invalid
    best = max(best, r - l + 1);
}
```
**Trap:** Works only when the window property is monotonic (shrinking never un-violates then re-violates weirdly). If not monotonic (e.g., "at most K distinct" is fine, but arbitrary conditions aren't), two pointers is unsafe — fall back to prefix sums + hashmap or segment tree.

---

### 2.8 Binary Search on Answer

**Trigger:** "Minimize the maximum X" / "maximize the minimum X" / "find smallest value satisfying predicate P" where P is monotonic (false...false,true...true).

```cpp
int lo = LOW, hi = HIGH, ans = hi;
while (lo <= hi) {
    int mid = lo + (hi - lo) / 2;
    if (feasible(mid)) { ans = mid; hi = mid - 1; }   // looking for smallest feasible
    else lo = mid + 1;
}
```
**Trap:** Verify monotonicity of `feasible()` before committing — this is the #1 reason binary-search-on-answer gives WA. Also watch for `lo+hi` overflow (use `lo + (hi-lo)/2`).

---

## 3. Graph Algorithms

### 3.1 Dijkstra (non-negative weights)

```cpp
vector<long long> dijkstra(int src, int n, vector<vector<pair<int,int>>>& adj) {
    vector<long long> dist(n, LLONG_MAX);
    priority_queue<pair<long long,int>, vector<pair<long long,int>>, greater<>> pq;
    dist[src] = 0; pq.push({0, src});
    while (!pq.empty()) {
        auto [d, u] = pq.top(); pq.pop();
        if (d > dist[u]) continue;                 // stale entry, skip
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
**Complexity:** O((V+E) log V). **Trap:** Must use `greater<>` for min-heap (default `priority_queue` is max-heap). Must have the stale-entry check (`if (d > dist[u]) continue`) — skipping it doesn't break correctness but can blow up complexity/TLE.

### 3.2 Bellman-Ford
**Trigger:** Negative edge weights, or need to **detect negative cycle**.
```cpp
bool bellmanFord(int n, int src, vector<array<int,3>>& edges, vector<long long>& dist) {
    dist.assign(n, LLONG_MAX); dist[src] = 0;
    for (int i = 0; i < n - 1; i++)
        for (auto& [u, v, w] : edges)
            if (dist[u] != LLONG_MAX && dist[u] + w < dist[v]) dist[v] = dist[u] + w;
    for (auto& [u, v, w] : edges)                 // one more pass = negative cycle check
        if (dist[u] != LLONG_MAX && dist[u] + w < dist[v]) return false; // neg cycle exists
    return true;
}
```
**Complexity:** O(V·E).

### 3.3 Floyd-Warshall
**Trigger:** All-pairs shortest path, N ≤ ~400–500.
```cpp
for (int k = 0; k < n; k++)
    for (int i = 0; i < n; i++)
        for (int j = 0; j < n; j++)
            if (dist[i][k] < INF && dist[k][j] < INF)
                dist[i][j] = min(dist[i][j], dist[i][k] + dist[k][j]);
```
**Complexity:** O(V^3). **Trap:** `k` must be the OUTERMOST loop — common mistake to swap loop order, which silently gives wrong answers.

### 3.4 0-1 BFS
**Trigger:** Edge weights only 0 or 1 — avoid Dijkstra's log factor.
```cpp
deque<int> dq; vector<int> dist(n, INT_MAX);
dist[src] = 0; dq.push_back(src);
while (!dq.empty()) {
    int u = dq.front(); dq.pop_front();
    for (auto [v, w] : adj[u]) {
        if (dist[u] + w < dist[v]) {
            dist[v] = dist[u] + w;
            if (w == 0) dq.push_front(v); else dq.push_back(v);
        }
    }
}
```

### 3.5 MST — Kruskal vs Prim
**Kruskal** (edge-list based, needs DSU): sort edges by weight, greedily add if it doesn't form a cycle.
```cpp
sort(edges.begin(), edges.end()); // by weight
DSU dsu(n); long long cost = 0;
for (auto& [w, u, v] : edges)
    if (dsu.unite(u, v)) cost += w;
```
**Complexity:** O(E log E).

**Prim** (adjacency based, good for dense graphs):
```cpp
vector<bool> inMST(n, false);
priority_queue<pair<int,int>, vector<pair<int,int>>, greater<>> pq;
pq.push({0, 0}); long long cost = 0;
while (!pq.empty()) {
    auto [w, u] = pq.top(); pq.pop();
    if (inMST[u]) continue;
    inMST[u] = true; cost += w;
    for (auto [v, wt] : adj[u]) if (!inMST[v]) pq.push({wt, v});
}
```
**Complexity:** O(E log V) with heap.

### 3.6 Topological Sort — Kahn's vs DFS
**Kahn's (BFS, indegree based)** — easier to detect cycles (if processed count < n → cycle exists):
```cpp
vector<int> indeg(n, 0);
for (auto& [u, v] : edges) indeg[v]++;
queue<int> q;
for (int i = 0; i < n; i++) if (indeg[i] == 0) q.push(i);
vector<int> order;
while (!q.empty()) {
    int u = q.front(); q.pop(); order.push_back(u);
    for (int v : adj[u]) if (--indeg[v] == 0) q.push(v);
}
bool hasCycle = (order.size() != n);
```
**DFS-based** — push node to a stack on post-order exit, reverse at the end. Useful when you naturally need post-order anyway (e.g., combined with DP on DAG).

### 3.7 Cycle Detection
- **Undirected:** DSU (edge connecting two already-same-component nodes = cycle) OR DFS with parent-tracking (visiting a visited non-parent node = cycle).
- **Directed:** DFS with 3 colors (white/gray/black) — a back-edge to a **gray** node means cycle. (Visiting a black node is fine.)

### 3.8 Bipartite Check
BFS/DFS 2-coloring; if any edge connects same-colored nodes → not bipartite. O(V+E).

### 3.9 LCA — Binary Lifting
**Trigger:** Many ancestor/LCA queries on a static tree.
```cpp
int LOG = 20;
vector<vector<int>> up(LOG, vector<int>(n));
vector<int> depth(n);
// up[0][v] = parent[v], filled via DFS/BFS
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
**Complexity:** O(log n) per query after O(n log n) preprocessing.

### 3.10 Euler Tour (flatten tree → array)
**Trigger:** Subtree sum/count queries → convert to range queries on `[tin[v], tout[v]]` using Fenwick/Segment Tree.
```cpp
int timer = 0;
vector<int> tin(n), tout(n);
void dfs(int u, int p) {
    tin[u] = timer++;
    for (int v : adj[u]) if (v != p) dfs(v, u);
    tout[u] = timer - 1;   // inclusive range covering entire subtree
}
// v is ancestor of u  <=>  tin[v] <= tin[u] && tout[u] <= tout[v]
```

### 3.11 SCC (brief — Kosaraju)
Two DFS passes: (1) DFS original graph, push to stack on finish; (2) DFS transposed graph in stack-popped order, each DFS tree = one SCC. O(V+E). Used for condensing graphs into DAGs (2-SAT, dependency problems).

---

## 4. Dynamic Programming Patterns

### 4.1 Bitmask DP
**Trigger:** N ≤ ~20, "assign/visit all items", TSP-style, "partition into groups" with small set size.
```cpp
// dp[mask] = min cost to have visited set `mask`, ending anywhere optimal
vector<vector<int>> dp(1 << n, vector<int>(n, INF));
dp[1][0] = 0;
for (int mask = 1; mask < (1 << n); mask++)
    for (int u = 0; u < n; u++) {
        if (!(mask & (1 << u)) || dp[mask][u] == INF) continue;
        for (int v = 0; v < n; v++) {
            if (mask & (1 << v)) continue;
            int nmask = mask | (1 << v);
            dp[nmask][v] = min(dp[nmask][v], dp[mask][u] + cost[u][v]);
        }
    }
```
**Complexity:** O(2^n · n^2) typical.

**Iterating submasks of a mask** (for "assign subset of items to group" DP):
```cpp
for (int sub = mask; sub > 0; sub = (sub - 1) & mask) {
    // process sub
}
// total across all masks: O(3^n)
```

### 4.2 Digit DP
**Trigger:** "count numbers in [L,R] with property P depending on digits" (digit sum, no repeated digit, k-th digit constraints).
```cpp
// dp[pos][tight][state] — memoize on (pos, state) when !tight
long long dp[20][2][STATE];
memset(dp, -1, sizeof dp);
function<long long(int,bool,int)> solve = [&](int pos, bool tight, int state) -> long long {
    if (pos == (int)digits.size()) return validState(state);
    if (!tight && dp[pos][tight][state] != -1) return dp[pos][tight][state];
    int limit = tight ? digits[pos] : 9;
    long long res = 0;
    for (int d = 0; d <= limit; d++)
        res += solve(pos + 1, tight && (d == limit), transition(state, d));
    if (!tight) dp[pos][tight][state] = res;
    return res;
};
```
**Trap:** Don't memoize when `tight == true` (state space collapses incorrectly — that path is bound to the specific prefix of the number).

### 4.3 LIS in O(n log n) — Patience Sorting
```cpp
vector<int> tails;
for (int x : a) {
    auto it = lower_bound(tails.begin(), tails.end(), x);   // strictly increasing LIS
    if (it == tails.end()) tails.push_back(x);
    else *it = x;
}
int lisLength = tails.size();
// For non-decreasing (allow equal), use upper_bound instead of lower_bound.
```
**Trap:** `lower_bound` → strictly increasing subsequence. `upper_bound` → non-decreasing. This single swap is the most common LIS bug.

### 4.4 Tree DP
**Trigger:** "Max independent set on tree", "diameter", "sum over subtrees" — DFS post-order, combine children's DP values at each node.
```cpp
// classic: max weight independent set on tree
pair<long long,long long> dfs(int u, int p) { // {take u, skip u}
    long long take = weight[u], skip = 0;
    for (int v : adj[u]) if (v != p) {
        auto [t, s] = dfs(v, u);
        take += s;
        skip += max(t, s);
    }
    return {take, skip};
}
```

### 4.5 Knapsack Variants (quick recall)
- **0/1 knapsack:** iterate items outer, capacity **inner loop descending** (to avoid reusing item).
- **Unbounded knapsack:** capacity loop **ascending** (reuse allowed).
- **Bounded (each item count Ci):** binary/power-of-two split into O(log Ci) "super items", then 0/1 knapsack. Or sliding window deque optimization if needed.

### 4.6 Interval DP
**Trigger:** "Merge/partition a sequence/string into optimal cost", matrix chain multiplication, burst balloons, palindrome partitioning cost.
```cpp
for (int len = 2; len <= n; len++)
    for (int i = 0; i + len - 1 < n; i++) {
        int j = i + len - 1;
        for (int k = i; k < j; k++)
            dp[i][j] = min(dp[i][j], dp[i][k] + dp[k+1][j] + cost(i, k, j));
    }
```
**Complexity:** O(n^3) typically. Iterate by increasing length, not by i/j directly.

---

## 5. Math Tricks

### 5.1 Fast Power / Modular Exponentiation
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

### 5.2 Modular Inverse (Fermat, when mod is prime)
```cpp
long long modInverse(long long a, long long mod) { return power(a, mod - 2, mod); }
```
**Trap:** Fermat's method only works when `mod` is prime. Otherwise use extended Euclidean algorithm.

### 5.3 nCr mod p (precompute factorials)
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

### 5.4 Sieve of Eratosthenes + Smallest Prime Factor
```cpp
vector<int> spf(MAXN);              // smallest prime factor
void sieve() {
    iota(spf.begin(), spf.end(), 0);
    for (int i = 2; (long long)i * i < MAXN; i++)
        if (spf[i] == i)
            for (int j = i * i; j < MAXN; j += i)
                if (spf[j] == j) spf[j] = i;
}
// factorize n in O(log n) using spf
vector<int> factorize(int n) {
    vector<int> f;
    while (n > 1) { f.push_back(spf[n]); n /= spf[n]; }
    return f;
}
```

### 5.5 GCD / LCM / Extended Euclid
```cpp
long long gcd(long long a, long long b) { return b ? gcd(b, a % b) : a; }   // or __gcd(a,b)
long long lcm(long long a, long long b) { return a / gcd(a, b) * b; }       // divide first! avoid overflow

// extended euclid: finds x,y such that a*x + b*y = gcd(a,b)
long long extgcd(long long a, long long b, long long& x, long long& y) {
    if (b == 0) { x = 1; y = 0; return a; }
    long long x1, y1;
    long long g = extgcd(b, a % b, x1, y1);
    x = y1; y = x1 - (a / b) * y1;
    return g;
}
```

### 5.6 Bit Tricks
```cpp
__builtin_popcount(x)         // count set bits (int)
__builtin_popcountll(x)       // for long long
__builtin_ctz(x)              // count trailing zeros (= position of lowest set bit)
__builtin_clz(x)              // count leading zeros
x & (-x)                      // isolates lowest set bit (lowbit)
x & (x - 1)                   // removes lowest set bit
x | (x + 1)                   // sets lowest unset bit (occasionally useful)
```

### 5.7 Matrix Exponentiation (for linear recurrences with huge n)
Represent recurrence as matrix `M`; answer = `M^n` applied to base vector. Use fast power on matrices (multiply is O(k^3) for k×k matrix). Trigger: "n up to 10^9 or 10^18" + linear recurrence (Fibonacci-like).

---

## 6. String Algorithms

### 6.1 KMP (pattern matching, O(n+m))
```cpp
vector<int> computeLPS(string& pat) {
    int m = pat.size(); vector<int> lps(m, 0);
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
LPS array itself is also useful standalone for "shortest repeating unit" / "longest proper prefix that is also suffix" questions.

### 6.2 Z-function
`z[i]` = length of longest substring starting at `i` that matches a prefix of the string.
```cpp
vector<int> zFunction(string& s) {
    int n = s.size(); vector<int> z(n, 0);
    int l = 0, r = 0;
    for (int i = 1; i < n; i++) {
        if (i < r) z[i] = min(r - i, z[i - l]);
        while (i + z[i] < n && s[z[i]] == s[i + z[i]]) z[i]++;
        if (i + z[i] > r) { l = i; r = i + z[i]; }
    }
    return z;
}
// pattern search: run on (pattern + '#' + text), matches where z[i] == pattern.size()
```

### 6.3 Polynomial String Hashing
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
long long getHash(int l, int r) { // [l, r], 0-indexed inclusive
    long long h = (prefHash[r+1] - prefHash[l] * powBase[r-l+1]) % MOD;
    return (h + MOD) % MOD;
}
```
**Trap:** Use **double hashing** (two different mod/base pairs) for OA-safe collision resistance if the problem is adversarial or has many string comparisons — single hash can be broken/collide.

### 6.4 Manacher (all palindromic substrings, O(n)) — brief
Transform string with separators (`#`) to handle even/odd uniformly, then expand using previously computed radius mirror trick. Usually only needed if O(n^2) expand-around-center TLEs (n > ~5000). Use expand-around-center (O(n^2)) by default — simpler and usually good enough for OA constraints; reach for Manacher only if n is large (10^5+).

---

## 7. Useful STL Constructs (the syntax you actually forget)

### 7.1 Binary search utilities
```cpp
lower_bound(v.begin(), v.end(), x)   // first element >= x
upper_bound(v.begin(), v.end(), x)   // first element > x
// index form:
int idx = lower_bound(v.begin(), v.end(), x) - v.begin();
bool exists = binary_search(v.begin(), v.end(), x);

// custom comparator (vector must be sorted w.r.t. same comparator!)
lower_bound(v.begin(), v.end(), x, [](int a, int b){ return a > b; }); // for descending vector

// on struct/pair by a specific field:
lower_bound(v.begin(), v.end(), make_pair(x, INT_MIN));  // trick: find first pair.first >= x
```

### 7.2 priority_queue (min-heap, custom comparator, pairs)
```cpp
priority_queue<int> maxHeap;                                        // default: max-heap
priority_queue<int, vector<int>, greater<int>> minHeap;              // min-heap

// custom comparator via struct (needed for complex sort logic)
struct Cmp {
    bool operator()(const pair<int,int>& a, const pair<int,int>& b) {
        return a.second > b.second;   // min-heap by .second
    }
};
priority_queue<pair<int,int>, vector<pair<int,int>>, Cmp> pq;

// lambda comparator (needs decltype since PQ template needs a type)
auto cmp = [](pair<int,int> a, pair<int,int> b) { return a.second > b.second; };
priority_queue<pair<int,int>, vector<pair<int,int>>, decltype(cmp)> pq2(cmp);
```
**Trap:** priority_queue comparator `return a < b` behaves like `operator<` — returning true means "a has LOWER priority than b" (opposite of `sort` intuition). If confused, just remember: `greater<int>` = min-heap.

### 7.3 set / multiset / map essentials
```cpp
set<int> s;
s.insert(x); s.erase(x);                 // erase(x) removes ALL x in set (fine, only one copy anyway)
auto it = s.find(x);                      // s.end() if not found
s.count(x);                               // 0 or 1 for set
auto it2 = s.lower_bound(x);              // first element >= x (built-in, O(log n) — faster than std::lower_bound on set!)

multiset<int> ms;
ms.insert(x);
ms.erase(ms.find(x));                     // erase ONE occurrence (ms.erase(x) removes ALL copies — common bug!)

map<int,int> mp;
mp[x]++;                                   // creates entry with 0 if absent, then increments
if (mp.find(x) == mp.end()) ...            // safe existence check (avoid mp[x] side-effect when just checking)
for (auto& [k, v] : mp) ...                // structured bindings, sorted by key automatically

// erase while iterating (erase-remove idiom / safe map iteration)
for (auto it = mp.begin(); it != mp.end(); ) {
    if (shouldErase(it)) it = mp.erase(it);
    else ++it;
}
```
**Trap:** `multiset::erase(value)` deletes **every** matching element — O(count) — use `erase(iterator)` to delete just one.

### 7.4 unordered_map / unordered_set caveats
- O(1) average, but **O(n) worst case** — can be adversarially attacked or hit hash collisions with certain inputs (notably `pair`/custom keys need a custom hash).
- No built-in hash for `pair<int,int>` — must supply one:
```cpp
struct pair_hash {
    size_t operator()(const pair<int,int>& p) const {
        return hash<long long>()(((long long)p.first << 32) ^ (unsigned int)p.second);
    }
};
unordered_map<pair<int,int>, int, pair_hash> mp;
```
- If TLE-ing mysteriously on `unordered_map` with large input, switch to `map` — sometimes it's actually faster in practice due to hash collision attacks on competitive judges, or reserve buckets: `mp.reserve(n); mp.max_load_factor(0.25);`

### 7.5 deque
```cpp
deque<int> dq;
dq.push_front(x); dq.push_back(x);
dq.pop_front(); dq.pop_back();
dq.front(); dq.back();
dq[i];                       // random access supported, O(1)
```

### 7.6 pair / tuple / structured bindings
```cpp
pair<int,int> p = {1, 2};
auto [a, b] = p;                                  // structured binding
tuple<int,int,int> t = {1, 2, 3};
auto [x, y, z] = t;
get<0>(t); get<1>(t);                              // access by index

// sorting a vector of tuples/pairs sorts lexicographically by default
sort(v.begin(), v.end());                          // pairs/tuples compare field-by-field automatically
```

### 7.7 Custom comparators & sort
```cpp
sort(v.begin(), v.end(), [](const auto& a, const auto& b) { return a.second < b.second; });

// sort indices by value in another array
vector<int> idx(n); iota(idx.begin(), idx.end(), 0);
sort(idx.begin(), idx.end(), [&](int i, int j) { return a[i] < a[j]; });

// stable_sort when relative order of equal elements must be preserved
stable_sort(v.begin(), v.end(), cmp);
```

### 7.8 Other frequently-forgotten STL
```cpp
next_permutation(v.begin(), v.end());              // generates permutations in-place, returns false when exhausted (needs sorted start for ALL perms)
__gcd(a, b);                                         // built-in gcd
accumulate(v.begin(), v.end(), 0LL);                 // sum (careful: 0LL not 0, to avoid overflow on ints)
*max_element(v.begin(), v.end());
*min_element(v.begin(), v.end());
reverse(v.begin(), v.end());
unique(v.begin(), v.end());                          // removes CONSECUTIVE duplicates only — sort first!
v.erase(unique(v.begin(), v.end()), v.end());        // the full dedup idiom
count(v.begin(), v.end(), x);
distance(v.begin(), it);
fill(v.begin(), v.end(), val);
memset(arr, 0, sizeof(arr));                         // only safe for 0 / -1 (byte pattern), NOT arbitrary ints
bitset<32> b(x);   b.count();   b.to_ulong();        // fixed-size bit manipulation, fast constant factor
__int128                                              // for overflow-safe big multiplication (print manually, no cin/cout support)
```

### 7.9 PBDS ordered_set (order statistics — if allowed / GNU extension)
```cpp
#include <ext/pb_ds/assoc_container.hpp>
#include <ext/pb_ds/tree_policy.hpp>
using namespace __gnu_pbds;
typedef tree<int, null_type, less<int>, rb_tree_tag, tree_order_statistics_node_update> ordered_set;

ordered_set os;
os.insert(x);
os.order_of_key(x);     // count of elements strictly less than x
*os.find_by_order(k);   // k-th smallest element (0-indexed)
```
**Trap:** Only works with GNU G++ compiler (usually fine on judges, but verify the OA platform supports `<ext/...>` headers before relying on it — have a Fenwick-over-compressed-ranks fallback ready).

### 7.10 Lambda syntax quick reference
```cpp
auto f = [](int x) { return x * 2; };
auto g = [&](int x) { return x + capturedVar; };       // capture by reference
auto h = [=](int x) { return x + capturedVar; };       // capture by value
auto rec = [&](auto&& self, int n) -> int {             // recursive lambda
    if (n <= 1) return 1;
    return n * self(self, n - 1);
};
function<int(int)> fib = [&](int n) { return n < 2 ? n : fib(n-1) + fib(n-2); }; // alt recursive style
```

---

## 8. Confused-Technique Comparison Tables

### Shortest Path
| Situation | Algorithm | Complexity |
|---|---|---|
| Non-negative weights, single source | Dijkstra | O((V+E) log V) |
| Negative weights allowed / need cycle detection | Bellman-Ford | O(V·E) |
| All-pairs, small V (≤ ~400-500) | Floyd-Warshall | O(V^3) |
| Weights are only 0/1 | 0-1 BFS | O(V+E) |
| Unweighted | plain BFS | O(V+E) |
| DAG (no cycles) | Topological order + relax | O(V+E) — fastest option |

### Range Query Structures
| Need | Structure |
|---|---|
| Static array, range min/max/gcd, no updates | Sparse Table (O(1) query) |
| Point update + range sum | Fenwick Tree (simplest, small constant) |
| Point update + range min/max | Segment Tree |
| Range update + range query | Segment Tree with lazy propagation (or 2×Fenwick for sum-only) |
| Range update + point query only | Fenwick over difference array |

### MST
| Situation | Use |
|---|---|
| Sparse graph, edge list given | Kruskal (sort + DSU) |
| Dense graph, adjacency matrix/list | Prim (heap-based) |

### set vs unordered_set vs multiset vs vector+sort
| Need | Container |
|---|---|
| Sorted order maintained, unique elements, log n ops | `set` |
| No order needed, fastest average lookup | `unordered_set` |
| Duplicates allowed, sorted | `multiset` |
| Build once, query many times, no further inserts | `vector` + `sort` + `binary_search`/`lower_bound` (fastest, best cache locality) |
| Need k-th order statistic dynamically | PBDS `ordered_set` |

### DP: Memoization vs Tabulation
| Situation | Prefer |
|---|---|
| Not all states are reachable/needed | Memoization (top-down) — computes only needed states |
| Need to optimize space (rolling array) | Tabulation (bottom-up) — easier to control array reuse |
| Recursion depth is large (risk of stack overflow) | Tabulation |
| Complex state transitions, easier to reason recursively | Memoization |

### Toposort: Kahn's vs DFS
| Situation | Prefer |
|---|---|
| Need explicit cycle detection with count check | Kahn's (compare processed count to n) |
| Already doing DFS for something else (e.g. tree DP on DAG) | DFS-based (post-order stack) |

### KMP vs Z-function vs Hashing
| Situation | Prefer |
|---|---|
| Single pattern search in text | KMP or Z-function (both O(n+m), pick whichever you code faster) |
| Need to compare MANY substrings for equality quickly | Hashing (O(1) compare after O(n) build) |
| Need exact guaranteed correctness (no collision risk) | KMP/Z (deterministic) over single hashing |
| Multiple pattern matching | Consider Aho-Corasick if truly needed (rare in OAs) |

---

## 9. Common OA Traps (things that cause WA/TLE even when logic is correct)

- **Integer overflow:** default `int` overflows past ~2.1×10^9. If products of up to two ~10^5 numbers, or any sum that could exceed 2×10^9, use `long long` everywhere including intermediate computations, not just the final answer variable.
- **`(a - b) % m` going negative** in C++ (unlike Python) — always do `((a - b) % m + m) % m`.
- **Global vs local array size:** declaring large arrays inside functions can blow the stack; prefer global/static arrays or `vector` for anything > ~10^4 elements.
- **Reading input speed:** for large inputs, add `ios_base::sync_with_stdio(false); cin.tie(nullptr);` at the very top of `main`.
- **`endl` vs `\n`:** `endl` flushes the buffer every time — use `\n` for heavy output to avoid TLE from I/O.
- **Off-by-one in binary search bounds** — always sanity check with a 1-element and 0-match case mentally.
- **Mutating a container while iterating it** without using the returned iterator from `erase`.
- **Recursion depth limits** — deep recursion (e.g. DFS on a skewed tree/graph with 10^5 nodes) can stack-overflow; convert to iterative DFS/BFS if `n` is large and the graph could be a long chain.
- **Uninitialized `visited`/`dist` arrays across multiple test cases** — if the OA has multiple test cases in one run, remember to reset global state each time (a classic silent bug).
- **Assuming input is 1-indexed or 0-indexed without checking constraints carefully.**
- **`vector<vector<int>> adj(n)` sizing mismatch** — using 1-indexed nodes but sizing adjacency list for `n` instead of `n+1`.
- **Floating point comparisons** — never use `==` for doubles; use `abs(a-b) < 1e-9`, and avoid floating point in loop conditions if an integer formulation exists.
- **Assuming greedy works without proving the exchange argument** — a suspiciously "obvious" greedy is the #1 source of subtly wrong solutions in OAs. If in doubt and time allows, stress-test against brute force mentally on a small example.
- **Multiset erase-all bug** (see 7.3) — extremely common silent bug under time pressure.
- **Segment tree / Fenwick array not sized/reset between multiple test cases within the same run.**

---

## 10. Final 15–30 Minute Revision (glance at this right before the OA)

**Recognition reflexes:**
- Range queries + updates → Fenwick (sum) / Segment Tree (min/max or lazy range updates) / Sparse Table (static, no updates).
- Connectivity/grouping → DSU.
- Next greater/smaller, sliding window max → Monotonic stack/deque.
- "Minimize the max" / "smallest X satisfying condition" → Binary search on answer (check monotonicity first!).
- Shortest path: non-negative → Dijkstra; negative → Bellman-Ford; all-pairs small N → Floyd-Warshall; 0/1 weights → 0-1 BFS.
- Small N (≤20) subset problems → Bitmask DP.
- XOR max pair/subarray → Trie on bits.
- k-th order stat dynamically → PBDS ordered_set, or Fenwick over compressed ranks.

**Syntax to not fumble:**
- `lower_bound`/`upper_bound`: lower = first `>=`, upper = first `>`. On sets, use `s.lower_bound()` (member function), NOT `std::lower_bound(s.begin(),...)` (O(n) on set iterators!).
- `priority_queue<int, vector<int>, greater<int>>` = min-heap.
- `multiset::erase(iterator)` = one element; `erase(value)` = all matching elements.
- LIS strictly increasing → `lower_bound`; non-decreasing allowed → `upper_bound`.
- `((a % m) + m) % m` for negative mod safety.
- `ios_base::sync_with_stdio(false); cin.tie(nullptr);` + use `\n` not `endl`.
- Fenwick is 1-indexed. Floyd-Warshall: `k` loop must be outermost.
- `long long` for anything that could exceed ~2×10^9 — when unsure, just use `long long`.

**Before submitting each solution:**
1. Check constraints → decide complexity budget (10^8 ops/sec rule of thumb).
2. Re-verify off-by-one in loop bounds and binary search.
3. Re-check overflow on any multiplication/summation.
4. If multiple test cases: confirm all global/static structures are reset.
5. Mentally test on the smallest possible edge case (n=1, empty input, all-equal elements, single node graph).

Good luck on the InMobi OA.