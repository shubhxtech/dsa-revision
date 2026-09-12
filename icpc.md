# ICPC India Prelims — Complete Preparation Guide

> **Who this is for:** You know the algorithms. You've done Striver's sheet. You've solved CF problems. Your real gap is *"I know DSU exists but I couldn't spot that this problem needs it."* This guide fixes that.

---

## Table of Contents

| # | Section |
|---|---------|
| 0 | [What ICPC India Prelims Actually Look Like](#0-what-icpc-india-prelims-actually-look-like) |
| 1 | [Priority Tiers — Where to Spend Your Time](#1-priority-tiers) |
| 2 | [The Recognition Checklist](#2-the-recognition-checklist) |
| 3 | [DSU — Union Find](#3-dsu--union-find) |
| 4 | [Graphs — BFS / DFS / Shortest Paths](#4-graphs--bfs--dfs--shortest-paths) |
| 5 | [Greedy + Sorting](#5-greedy--sorting) |
| 6 | [Binary Search (on answer + on array)](#6-binary-search) |
| 7 | [DP — Fundamentals to Advanced](#7-dp--fundamentals-to-advanced) |
| 8 | [Trees — Rooted DP, LCA, Rerooting](#8-trees) |
| 9 | [Number Theory + Combinatorics](#9-number-theory--combinatorics) |
| 10 | [Bit Manipulation + XOR tricks](#10-bit-manipulation--xor-tricks) |
| 11 | [Prefix Sums + Difference Arrays](#11-prefix-sums--difference-arrays) |
| 12 | [Segment Tree + Fenwick Tree (BIT)](#12-segment-tree--fenwick-tree) |
| 13 | [Two Pointers + Sliding Window](#13-two-pointers--sliding-window) |
| 14 | [Constructive Algorithms](#14-constructive-algorithms) |
| 15 | [Ad-hoc + Invariants + Parity](#15-ad-hoc--invariants--parity) |
| 16 | [Game Theory (Nim + Grundy)](#16-game-theory) |
| 17 | [String Algorithms (KMP, Z, Hashing)](#17-string-algorithms) |
| 18 | [Practice Strategy](#18-practice-strategy) |
| 19 | [Quick Reference — If X then Y](#19-quick-reference) |

---

## 0. What ICPC India Prelims Actually Look Like

### Format
- **6 problems, 2.5–3 hours**, all teams on the same problem set
- Scoring: ICPC-style penalty (all-or-nothing per problem, time penalty for wrong submissions)
- Language: C++17 on Codeforces (historically GNU G++17 7.3.0)
- Difficulty spread: Problem A is a near-freebie (1000–1400 CF rating), Problems E–F are genuinely hard (2200+), Problems B–D are where most teams' rank is decided (1600–2000)

### What actually appeared (2023–2025)

| Year | Problem | Type |
|---|---|---|
| 2025–26 | How many? | Constructive / Math |
| 2025–26 | Pseudo Palindrome | Greedy + Sorting (exchange argument) |
| 2025–26 | XOR LCM | Number Theory + XOR |
| 2025–26 | Make Empty | DP / Greedy |
| 2025–26 | Counting Is Fun | Combinatorics / counting |
| 2024–25 | Collisions | Ad-hoc / Invariant |
| 2024–25 | Points and Threads | Graph modeling / DSU |
| 2024–25 | bincatmod | Bit manipulation + modular math |
| 2024–25 | Tree Construction | Constructive + Tree |
| 2023–24 | Equations | Weighted DSU |
| 2023–24 | Small Indices | DP with bounded state |
| 2023–24 | Yet Another GCD Problem | Number Theory |

**Key observations:**
1. DSU appears almost every year — often disguised as "pairwise constraint satisfaction"
2. Number theory + XOR/bit manipulation is a guaranteed problem every year
3. DP at this level is never textbook — there's always a twist (bounded state, bitmask, digit DP)
4. Advanced DS (segment tree beats, HLD, flows, FFT) essentially never decides a prelims result

---

## 1. Priority Tiers

### Tier 1 — Must be instant (write in under 10 min, no bugs)
- DSU (plain + weighted/parity variant)
- BFS / 0-1 BFS / Dijkstra
- Binary search on the answer
- Sorting + greedy with exchange argument
- Prefix sums + difference arrays
- Basic DP (1D knapsack, 2D grid DP, transition derivation)
- Modular arithmetic, fast exponentiation, nCr mod p
- Bit manipulation basics (XOR properties, bitmask iteration)
- Sieve of Eratosthenes

### Tier 2 — Strong fluency required
- Trees: rooted DP, LCA (binary lifting), rerooting
- Digit DP, bitmask DP (n ≤ 20)
- Monotone stack / deque DP optimization
- Number theory: GCD sieve, inclusion-exclusion, Euler totient
- Two pointers / sliding window
- BFS over augmented state (position + extra info)
- Constructive algorithms: small cases, parity/necessary conditions, greedy build order
- Basic segment tree + BIT (Fenwick)

### Tier 3 — Know the idea, implement if needed
- Segment tree with lazy propagation
- KMP / Z-function / rolling hash
- Basic game theory (Nim, Sprague-Grundy)
- Matrix exponentiation
- Convex hull

### Tier 4 — Learn after prelims (rarely decides prelims)
- Segment tree beats, persistent segment tree
- Suffix automaton/array, Aho-Corasick
- Max flow, min-cost flow
- FFT / NTT
- Heavy-light decomposition
- General half-plane intersection

---

## 2. The Recognition Checklist

Run this on every problem, in order, before writing a single line of code.

| Step | Ask | Points to |
|---|---|---|
| 1 | What are n and the time limit? | n≤20→bitmask DP, n≤500→O(n²) ok, n≤1e5→O(n log n), n≤1e7→linear/sieve |
| 2 | Are pairwise relations given one at a time? | DSU |
| 3 | Minimum steps/moves on a graph or grid? | BFS (0-1 BFS if weights 0/1, Dijkstra otherwise) |
| 4 | Optimize over choices with overlapping sub-problems? | DP |
| 5 | "Rearrange / reorder to minimize something"? | Sort first, then greedy |
| 6 | "Largest/smallest X such that condition holds"? | Binary search on X if condition is monotone |
| 7 | "Number of ways… mod 1e9+7"? | Counting DP or combinatorics |
| 8 | GCD, LCM, divisors, coprime pairs? | Divisor sieve, fix-the-GCD trick |
| 9 | XOR / bits / binary representation? | Per-bit decomposition, trie for max XOR |
| 10 | "Construct any valid arrangement"? | Small cases by hand → parity check → greedy order |
| 11 | Final answer seems independent of operation order? | Find the invariant, reduce to counting a local pattern |
| 12 | Graph + extra toggleable fact (keys, parity, fuel)? | BFS over (position, extra state) |

---

## 3. DSU — Union Find

### Recognition
- Pairwise relations processed one at a time: "a and b must be equal," "a and b are in the same group," "the XOR/sum/difference of a and b is fixed"
- "Is this set of constraints consistent?"
- "Count components" as edges arrive
- Keywords: **same group, merge, friends-of-friends, connected component, belong to the same team**

> **ICPC India signal:** Almost every year has a DSU problem, but it's disguised as constraint satisfaction — not "compute MST" or "build a graph." *Equations* (2023–24) is the clearest example: given `a XOR b = w` for pairs, determine if a consistent assignment exists.

### Plain DSU
```cpp
const int MAXN = 1e5 + 5;
int parent[MAXN], sz[MAXN];

void init(int n) {
    for (int i = 0; i < n; i++) parent[i] = i, sz[i] = 1;
}

int find(int x) {
    while (parent[x] != x) {
        parent[x] = parent[parent[x]];  // path halving
        x = parent[x];
    }
    return x;
}

bool unite(int a, int b) {
    a = find(a); b = find(b);
    if (a == b) return false;
    if (sz[a] < sz[b]) swap(a, b);
    parent[b] = a;
    sz[a] += sz[b];
    return true;
}
```

### Weighted (Parity/XOR) DSU
```cpp
// rel[v] = value[v] XOR value[find(v)] (swap XOR with +/- as needed)
int par[MAXN];
long long rel[MAXN];

void init(int n) {
    for (int i = 0; i < n; i++) par[i] = i, rel[i] = 0;
}

pair<int, long long> find(int v) {
    if (par[v] == v) return {v, 0};
    auto [root, r] = find(par[v]);
    par[v] = root;
    rel[v] ^= r;       // fold in parent's offset during path compression
    return {root, rel[v]};
}

// Returns false if enforcing "value[a] XOR value[b] = w" creates contradiction
bool unite(int a, int b, long long w) {
    auto [ra, xa] = find(a);
    auto [rb, xb] = find(b);
    if (ra == rb) return (xa ^ xb) == w;   // consistency check
    par[ra] = rb;
    rel[ra] = xa ^ xb ^ w;                 // derived to preserve equation globally
    return true;
}
```

> For sum constraints: replace `^` with `+`/`-`.
> For parity/bipartite: use `rel[v] = 0 or 1` (which side of bipartition), `rel[ra] = xa ^ xb ^ 1` to enforce "different side."

### Problem Set — DSU

| Problem | CF Link | Rating | What to practice |
|---|---|---|---|
| 200E — Tractor | https://codeforces.com/problemset/problem/200/E | 1900 | DSU on edges sorted by weight |
| 25D — Roads not only in Berland | https://codeforces.com/problemset/problem/25/D | 1700 | DSU tree reconstruction |
| 455C — Civilization | https://codeforces.com/problemset/problem/455/C | 1700 | DSU merging trees + diameter |
| 566D — Restructuring Company | https://codeforces.com/problemset/problem/566/D | 1700 | DSU with extra info per component |
| 1209G — Across the Universe | https://codeforces.com/problemset/problem/1209/G | 2000 | DSU + counting |
| 1131F — Asya and Kittens | https://codeforces.com/problemset/problem/1131/F | 1700 | Build tree using DSU |
| 1416C — XOR Inverse | https://codeforces.com/problemset/problem/1416/C | 2100 | XOR DSU concept |
| 1503C — Travelling Salesman and Special Numbers | https://codeforces.com/problemset/problem/1503/C | 1600 | Warms up before DSU on trees |
| 827F — Elections | https://codeforces.com/problemset/problem/827/F | 1900 | Weighted DSU |
| 722F — Циклы | https://codeforces.com/problemset/problem/722/F | 2100 | DSU cycle detection |

**Searchable tag:** https://codeforces.com/problemset?tags=dsu&order=BY_RATING_ASC

---

## 4. Graphs — BFS / DFS / Shortest Paths

### Recognition
- "Minimum number of moves/operations" on unweighted graph or grid → **BFS**
- Weights are only 0 or 1 → **0-1 BFS** (use deque, push front for 0-weight, back for 1-weight)
- Positive integer weights → **Dijkstra**
- Negative weights (no negative cycles) → **Bellman-Ford**
- All-pairs shortest paths → **Floyd-Warshall** (small n)

### BFS over augmented state
When the same physical position can be "different" depending on extra information (keys held, parity of steps, remaining fuel, toggle state), add that as a dimension:
- State = `(position, extra_info)`
- Number of states = `positions × possible values of extra_info`
- Use BFS/Dijkstra over this expanded state space

```cpp
// 0-1 BFS template
const int INF = 1e9;
int dist[MAXN];

void bfs01(int src, vector<pair<int,int>> adj[], int n) {
    fill(dist, dist + n, INF);
    dist[src] = 0;
    deque<int> dq;
    dq.push_back(src);

    while (!dq.empty()) {
        int u = dq.front();
        dq.pop_front();
        for (auto [v, w] : adj[u]) {
            if (dist[u] + w < dist[v]) {
                dist[v] = dist[u] + w;
                if (w == 0) dq.push_front(v);
                else dq.push_back(v);
            }
        }
    }
}
```

### Dijkstra (Striver style)
```cpp
const long long INF = 1e18;
vector<pair<int,int>> adj[MAXN];
long long dist[MAXN];

void dijkstra(int src, int n) {
    fill(dist, dist + n, INF);
    dist[src] = 0;
    priority_queue<pair<long long,int>, vector<pair<long long,int>>, greater<pair<long long,int>>> pq;
    pq.push({0, src});

    while (!pq.empty()) {
        long long d = pq.top().first;
        int u = pq.top().second;
        pq.pop();
        if (d > dist[u]) continue;
        for (int i = 0; i < adj[u].size(); i++) {
            int v = adj[u][i].first;
            long long w = adj[u][i].second;
            if (dist[u] + w < dist[v]) {
                dist[v] = dist[u] + w;
                pq.push({dist[v], v});
            }
        }
    }
}
```

### Problem Set — Graphs

| Problem | CF Link | Rating | What to practice |
|---|---|---|---|
| 1915G — Bicycles | https://codeforces.com/problemset/problem/1915/G | 1900 | Dijkstra with extra state |
| 1846F — Vlad and Cafes | https://codeforces.com/problemset/problem/1846/F | 1600 | Multi-source BFS |
| 1891E — Bruteforcers | https://codeforces.com/problemset/problem/1891/E | 1700 | BFS on implicit graph |
| 1811F — Is It Flower? | https://codeforces.com/problemset/problem/1811/F | 1700 | Graph modeling |
| 580C — Kefa and Park | https://codeforces.com/problemset/problem/580/C | 1500 | BFS/DFS on tree |
| 1272F — Two Bracket Sequences | https://codeforces.com/problemset/problem/1272/F | 2000 | BFS with augmented state |
| 1093G — Multidimensional Queries | https://codeforces.com/problemset/problem/1093/G | 2400 | Complex state BFS |
| 1503B — Grid Figure | https://codeforces.com/problemset/problem/1503/B | 1400 | Grid BFS |
| 1765E — Rectangular Congruence | https://codeforces.com/problemset/problem/1765/E | 2000 | Graph + math |
| 786C — Till I collapse | https://codeforces.com/problemset/problem/786/C | 2200 | Offline graph queries |

**0-1 BFS specifically:**
- 173B — https://codeforces.com/problemset/problem/173/B
- 590C — https://codeforces.com/problemset/problem/590/C

**Searchable tag:** https://codeforces.com/problemset?tags=shortest+paths&order=BY_RATING_ASC

---

## 5. Greedy + Sorting

### Recognition
- "Pair up / assign / schedule elements to minimize/maximize something"
- The word "rearrange" or "reorder" appears
- No overlapping subproblems → greedy (vs DP where you can't commit locally)
- "Pair elements so that [some aggregate property] is satisfied"

### The Exchange Argument
1. Sort by the most relevant key
2. Guess a pairing/ordering rule (adjacent after sort? smallest-with-largest? split-in-half?)
3. Prove: swap any two elements that violate the rule → the answer doesn't improve
4. If you can complete the exchange argument in ~2 minutes mentally → it's greedy. Otherwise → suspect DP.

**Common greedy patterns:**
- Match sorted lower half to sorted upper half (minimize max difference among pairs)
- Sort by deadline/ratio/ratio-of-ratio for scheduling
- Greedy from both ends (two-pointer greedy)
- Sort and greedily assign resources left-to-right

### Problem Set — Greedy + Sorting

| Problem | CF Link | Rating | What to practice |
|---|---|---|---|
| 455B — A Lot of Games | https://codeforces.com/problemset/problem/455/B | 1500 | Greedy game theory |
| 1360E — Polygon | https://codeforces.com/problemset/problem/1360/E | 1700 | Sort + greedy |
| 1203E — Boxers | https://codeforces.com/problemset/problem/1203/E | 1600 | Pairing greedy |
| 1553C — Penalty Kick | https://codeforces.com/problemset/problem/1553/C | 1500 | Simple greedy with a twist |
| 1601C — Optimal Insertion | https://codeforces.com/problemset/problem/1601/C | 2100 | D&C greedy |
| 1428D — Bouncing | https://codeforces.com/problemset/problem/1428/D | 1700 | Interval greedy |
| 1399D — Binary String To Palindrome | https://codeforces.com/problemset/problem/1399/D | 1500 | Greedy observation |
| 1463C — Busy Robot | https://codeforces.com/problemset/problem/1463/C | 1600 | Simulation + greedy |
| 1374D — Zero Remainder Array | https://codeforces.com/problemset/problem/1374/D | 1600 | Sort + greedy |
| 1526C2 — Max Binary Matrix | https://codeforces.com/problemset/problem/1526/C2 | 1800 | Greedy + observation |

**Searchable tag:** https://codeforces.com/problemset?tags=greedy,sortings&order=BY_RATING_ASC

---

## 6. Binary Search

### Two types

**Type 1 — Binary search on a sorted array/structure**
Classic: find first position where condition holds. O(log n).

**Type 2 — Binary search on the answer**
- Define `feasible(x)` = "can we achieve the goal with answer ≤ x (or ≥ x)?"
- Prove monotonicity: if `feasible(x)` is true, then `feasible(x+1)` is also true (or false)
- Binary search on `x` over the range of possible answers
- `feasible` itself is often a greedy or simulation

```cpp
// Binary search on answer — minimize maximum
long long lo = min_possible, hi = max_possible;
while (lo < hi) {
    long long mid = lo + (hi - lo) / 2;
    if (feasible(mid)) hi = mid;
    else lo = mid + 1;
}
// answer is lo
```

> **Pitfall:** Always be careful about whether you're minimizing or maximizing. For maximize: swap the `hi = mid` / `lo = mid + 1` logic. For non-integer binary search: use `while (hi - lo > 1e-9)` with `mid = (lo + hi) / 2.0`.

### Problem Set — Binary Search

| Problem | CF Link | Rating | What to practice |
|---|---|---|---|
| 1538C — Number of Ways | https://codeforces.com/problemset/problem/1538/C | 1400 | Classic binary search on array |
| 1538D — Another Problem | https://codeforces.com/problemset/problem/1538/D | 1700 | Binary search on answer |
| 1739C — Losing Ticket | https://codeforces.com/problemset/problem/1739/C | 1500 | Binary search + counting |
| 1462F — The Treasure of The Segments | https://codeforces.com/problemset/problem/1462/F | 1700 | Binary search on answer |
| 1650C — Weight of the System of Equations | https://codeforces.com/problemset/problem/1650/C | 1800 | Binary search with math |
| 1614C — Latest Deadline | https://codeforces.com/problemset/problem/1614/C | 1600 | Binary search + greedy |
| 1622D — Ants | https://codeforces.com/problemset/problem/1622/D | 1700 | Binary search on value |
| 1741F — Xana and Inversions | https://codeforces.com/problemset/problem/1741/F | 1900 | Binary search + segment tree |
| 702C — Cellular Network | https://codeforces.com/problemset/problem/702/C | 1500 | Classic binary search |
| 817C — Really Big Numbers | https://codeforces.com/problemset/problem/817/C | 1400 | Binary search on answer |

**Searchable tag:** https://codeforces.com/problemset?tags=binary+search&order=BY_RATING_ASC

---

## 7. DP — Fundamentals to Advanced

### How to derive a DP

1. **Define state in one sentence.** `dp[i]` = best answer using first `i` elements.
2. **Identify what information you need from "history" to make the next decision.** That's your state dimensions.
3. **Write the transition.** `dp[i] = f(dp[j], ...)` for some `j < i`.
4. **Check state size.** If it's too large, look for a bound: does the state grow multiplicatively each time it changes? If so, it changes at most `O(log(max))` times — compress the state.
5. **Order of computation.** Fill in increasing order of "smaller sub-problems first."

### The State-Bounding Trick (ICPC India style)
> If a DP dimension's value can only increase multiplicatively (e.g., Fibonacci-like growth), then at most `O(log(max_value))` distinct values of that dimension can be reached. Re-index by "how many times has it grown" instead of the raw value → dimension size collapses from n to ~60.

### Bitmask DP (n ≤ 20)
```cpp
// dp[mask] = best answer using elements in mask
// iterate all submasks: for (int sub = mask; sub > 0; sub = (sub-1) & mask)
int dp[1 << 20];
memset(dp, 0x3f, sizeof(dp));
dp[0] = 0;
for (int mask = 1; mask < (1 << n); mask++) {
    for (int i = 0; i < n; i++) {
        if (mask & (1 << i)) {
            dp[mask] = min(dp[mask], dp[mask ^ (1 << i)] + cost[i]);
        }
    }
}
```

### Digit DP
```cpp
// Count numbers in [0, N] satisfying some digit-by-digit property
// State: (current position, tight constraint, extra state)
// "tight" = whether all digits so far exactly match N's prefix
int memo[20][2][EXTRA];
// solve(pos, tight, extra_state)
// tight=1: next digit can be at most N[pos]
// tight=0: next digit can be 0-9 freely
```

### Problem Set — DP

**Fundamentals:**

| Problem | CF Link | Rating | What to practice |
|---|---|---|---|
| 189A — Cut Ribbon | https://codeforces.com/problemset/problem/189/A | 1300 | Unbounded knapsack |
| 698B — Fix It | https://codeforces.com/problemset/problem/698/B | 1100 | Simple 1D DP |
| 166E — Numberphile | https://codeforces.com/problemset/problem/166/E | 1700 | 2D DP |
| 1183E — Subsequence | https://codeforces.com/problemset/problem/1183/E | 2200 | Tree DP + merging |
| 1063F — String Journey | https://codeforces.com/problemset/problem/1063/F | 2400 | DP + hashing |

**Intermediate (crucial for ICPC):**

| Problem | CF Link | Rating | What to practice |
|---|---|---|---|
| 1547E — Air Conditioners | https://codeforces.com/problemset/problem/1547/E | 1500 | Multi-source DP |
| 1398D — Multigrid Game | https://codeforces.com/problemset/problem/1398/D | 1700 | DP + game theory |
| 1525E — Assimilation IV | https://codeforces.com/problemset/problem/1525/E | 1900 | DP + sorting |
| 1446D — Lena and Sequence | https://codeforces.com/problemset/problem/1446/D | 1700 | DP with bound |
| 1601D — Difficult Mountain | https://codeforces.com/problemset/problem/1601/D | 1700 | DP / greedy |
| 1556E — Equilibrium | https://codeforces.com/problemset/problem/1556/E | 1900 | DP + construction |
| 1814C — Unforgivable Cursed Code | https://codeforces.com/problemset/problem/1814/C | 1500 | DP observation |
| 817E — Ice cream coloring | https://codeforces.com/problemset/problem/817/E | 1700 | Graph + DP |

**Bitmask DP:**

| Problem | CF Link | Rating | What to practice |
|---|---|---|---|
| 1556D — Take a Guess | https://codeforces.com/problemset/problem/1556/D | 1700 | Bitmask + greedy |
| 1032E — The Tyrant | https://codeforces.com/problemset/problem/1032/E | 2100 | Bitmask DP |
| 1316E — Team Building | https://codeforces.com/problemset/problem/1316/E | 2100 | Bitmask DP on DAG |
| 1244E — Minimizing Difference | https://codeforces.com/problemset/problem/1244/E | 1900 | DP with bitmask state |

**Digit DP:**

| Problem | CF Link | Rating | What to practice |
|---|---|---|---|
| 55D — Beautiful numbers | https://codeforces.com/problemset/problem/55/D | 2200 | Classic digit DP |
| 276D — Little Girl and Maximum XOR | https://codeforces.com/problemset/problem/276/D | 1800 | Digit DP idea |
| 1036E — Covered Points | https://codeforces.com/problemset/problem/1036/E | 2100 | Counting with digit-style thinking |
| 628E — Zbigniew's Signature | https://codeforces.com/problemset/problem/628/E | 2100 | Digit DP |

**Searchable tags:**
- https://codeforces.com/problemset?tags=dp&order=BY_RATING_ASC
- https://codeforces.com/problemset?tags=dp,bitmasks&order=BY_RATING_ASC
- https://codeforces.com/problemset?tags=digit+dp&order=BY_RATING_ASC

---

## 8. Trees

### Key operations you must know
1. **Rooted tree DFS** — compute subtree sizes, depths, parents in O(n)
2. **Subtree DP** — `dp[node]` = answer for node's subtree; merge children's answers
3. **Rerooting** — compute the answer for every node as root in O(n) total (not O(n²))
4. **LCA (binary lifting)** — `up[v][k]` = 2^k-th ancestor of v; O(n log n) build, O(log n) per query
5. **Euler tour** — linearize the tree so subtree = contiguous segment → enables range queries on subtrees

### Subtree DP template
```cpp
// dp[u] = answer for subtree rooted at u
// Merge children one by one; DFS handles ordering automatically
long long dp[MAXN];
vector<int> adj[MAXN];

void dfs(int u, int par) {
    dp[u] = base_value;
    for (int v : adj[u]) {
        if (v == par) continue;
        dfs(v, u);
        dp[u] = merge(dp[u], dp[v]);  // define merge per problem
    }
}
```

### Problem Set — Trees

**Subtree DP:**

| Problem | CF Link | Rating | What to practice |
|---|---|---|---|
| 600E — Lomsat gelral | https://codeforces.com/problemset/problem/600/E | 2100 | DSU on tree / subtree DP |
| 1017E — The Supersonic | https://codeforces.com/problemset/problem/1017/E | 2100 | Tree DP |
| 1097G — Vladislav and a Tree | https://codeforces.com/problemset/problem/1097/G | 2000 | Tree DP + DSU |
| 1695D — Max GEQ Sum | https://codeforces.com/problemset/problem/1695/D | 1800 | Tree DP |
| 1585F — Non-equal Neighbors | https://codeforces.com/problemset/problem/1585/F | 2200 | Tree DP counting |
| 1539F — Strange Operating System | https://codeforces.com/problemset/problem/1539/F | 2100 | Tree structure DP |

**LCA + distance queries:**

| Problem | CF Link | Rating | What to practice |
|---|---|---|---|
| 1535E — Distance Learning | https://codeforces.com/problemset/problem/1535/E | 1900 | LCA + distance |
| 1328E — Tree Painting | https://codeforces.com/problemset/problem/1328/E | 2100 | LCA + counting |
| 1627F — Bessie's Snow Cow | https://codeforces.com/problemset/problem/1627/F | 2300 | Tree paths + LCA |
| 1611G — Doom | https://codeforces.com/problemset/problem/1611/G | 2200 | LCA + graph |
| 519E — A and B and Lecture Rooms | https://codeforces.com/problemset/problem/519/E | 2100 | LCA path queries |

**Easy tree warm-ups:**

| Problem | CF Link | Rating | What to practice |
|---|---|---|---|
| 580C — Kefa and Park | https://codeforces.com/problemset/problem/580/C | 1500 | DFS on tree |
| 766C — Mahmoud and Ehab | https://codeforces.com/problemset/problem/766/C | 1600 | Tree coloring DP |
| 1081C — Colorful Graph | https://codeforces.com/problemset/problem/1081/C | 1500 | DFS |
| 455C — Civilization | https://codeforces.com/problemset/problem/455/C | 1700 | Tree diameter + DSU |

**Searchable tags:**
- https://codeforces.com/problemset?tags=trees,dp&order=BY_RATING_ASC
- https://codeforces.com/problemset?tags=trees&order=BY_RATING_ASC

---

## 9. Number Theory + Combinatorics

### Core toolkit

**GCD / LCM:**
```cpp
int gcd(int a, int b) { return b ? gcd(b, a % b) : a; }
long long lcm(long long a, long long b) { return a / gcd(a, b) * b; }
```

**Modular exponentiation:**
```cpp
long long power(long long a, long long e, long long mod) {
    long long r = 1;
    a %= mod;
    while (e > 0) {
        if (e & 1) r = r * a % mod;
        a = a * a % mod;
        e >>= 1;
    }
    return r;
}
long long modinv(long long a, long long mod) { return power(a, mod - 2, mod); }
```

**Precompute nCr mod p:**
```cpp
const int MAXN = 2e5 + 5, MOD = 1e9 + 7;
long long fact[MAXN], inv_fact[MAXN];

void precompute() {
    fact[0] = 1;
    for (int i = 1; i < MAXN; i++) fact[i] = fact[i-1] * i % MOD;
    inv_fact[MAXN-1] = modinv(fact[MAXN-1], MOD);
    for (int i = MAXN-2; i >= 0; i--) inv_fact[i] = inv_fact[i+1] * (i+1) % MOD;
}

long long nCr(int n, int r) {
    if (r < 0 || r > n) return 0;
    return fact[n] * inv_fact[r] % MOD * inv_fact[n-r] % MOD;
}
```

**Sieve of Eratosthenes:**
```cpp
const int MAXV = 1e6 + 5;
bool is_prime[MAXV];
vector<int> primes;

void sieve() {
    fill(is_prime, is_prime + MAXV, true);
    is_prime[0] = is_prime[1] = false;
    for (int i = 2; i < MAXV; i++) {
        if (is_prime[i]) {
            primes.push_back(i);
            for (long long j = (long long)i*i; j < MAXV; j += i)
                is_prime[j] = false;
        }
    }
}
```

**Fix-the-GCD trick (count pairs with gcd = k):**
```cpp
// f[m] = count of elements divisible by m
// g[m] = count of pairs with GCD exactly m
// g[m] = C(f[m], 2) - sum of g[m*t] for t >= 2
vector<long long> countPairsByGCD(vector<int>& a, int MAXV) {
    vector<long long> f(MAXV + 1, 0), g(MAXV + 1, 0);
    for (int x : a) if (x <= MAXV) f[x]++;
    for (int m = 1; m <= MAXV; m++)
        for (int mult = 2*m; mult <= MAXV; mult += m)
            f[m] += f[mult];
    for (int m = MAXV; m >= 1; m--) {
        g[m] = f[m] * (f[m] - 1) / 2;
        for (int mult = 2*m; mult <= MAXV; mult += m)
            g[m] -= g[mult];
    }
    return g;
}
```

### Problem Set — Number Theory + Combinatorics

| Problem | CF Link | Rating | What to practice |
|---|---|---|---|
| 1A — Theatre Square | https://codeforces.com/problemset/problem/1/A | 1000 | Ceiling division |
| 1538C — Number of Ways | https://codeforces.com/problemset/problem/1538/C | 1400 | Counting with prefix sums |
| 1777C — Tonya and Buratino | https://codeforces.com/problemset/problem/1777/C | 1900 | GCD counting |
| 1748D — Permutation Restoration | https://codeforces.com/problemset/problem/1748/D | 1700 | Combinatorics |
| 1550D — Three Regions | https://codeforces.com/problemset/problem/1550/D | 1700 | Inclusion-exclusion |
| 1553D — Backspace | https://codeforces.com/problemset/problem/1553/D | 1400 | Counting |
| 1542E — Abnormal Permutations | https://codeforces.com/problemset/problem/1542/E | 2100 | Combinatorics + DP |
| 1559E — Permutation Bin | https://codeforces.com/problemset/problem/1559/E | 2100 | GCD + combinatorics |
| 1566D — Frog Traveler | https://codeforces.com/problemset/problem/1566/D | 1900 | Number theory + BFS |
| 1610D — Spell Check | https://codeforces.com/problemset/problem/1610/D | 1700 | Counting + modular |
| 1617F — Calling Center | https://codeforces.com/problemset/problem/1617/F | 1900 | Number theory |
| 1073E — Segment Sum | https://codeforces.com/problemset/problem/1073/E | 2100 | Digit-style counting |

**Searchable tags:**
- https://codeforces.com/problemset?tags=number+theory&order=BY_RATING_ASC
- https://codeforces.com/problemset?tags=combinatorics&order=BY_RATING_ASC
- https://codeforces.com/problemset?tags=number+theory,combinatorics&order=BY_RATING_ASC

---

## 10. Bit Manipulation + XOR Tricks

### Recognition
- Statement has: **XOR, binary representation, bits, powers of two, bincatmod**
- Small n (≤ 20–25) with "subset" → bitmask iteration
- "Max XOR of two elements" → trie over bits
- "Count subsets with XOR = k" → linear basis / Gaussian elimination
- "Binary concatenation mod m" → track running modular value while appending bits

### Bit Trie (max XOR pair/query)
```cpp
struct BitTrie {
    int nxt[2];
    BitTrie() { nxt[0] = nxt[1] = -1; }
};

vector<BitTrie> trie(1);
const int BITS = 30;

void insert(int x) {
    int cur = 0;
    for (int b = BITS; b >= 0; b--) {
        int bit = (x >> b) & 1;
        if (trie[cur].nxt[bit] == -1) {
            trie[cur].nxt[bit] = trie.size();
            trie.push_back(BitTrie());
        }
        cur = trie[cur].nxt[bit];
    }
}

int maxXorWith(int x) {
    int cur = 0, res = 0;
    for (int b = BITS; b >= 0; b--) {
        int bit = (x >> b) & 1, want = 1 - bit;
        if (trie[cur].nxt[want] != -1) {
            res |= (1 << b);
            cur = trie[cur].nxt[want];
        } else {
            cur = trie[cur].nxt[bit];
        }
    }
    return res;
}
```

### Linear Basis (XOR basis for a set of numbers)
```cpp
// Maintains a basis where any XOR of a subset of the original array
// can be expressed as XOR of a subset of the basis
long long basis[64];
int sz = 0;

bool insert(long long x) {
    for (int i = 0; i < sz; i++) x = min(x, x ^ basis[i]);
    if (x == 0) return false;  // x is already spanned by current basis
    basis[sz++] = x;
    return true;
}

long long maxXOR() {
    long long res = 0;
    for (int i = sz - 1; i >= 0; i--)
        res = max(res, res ^ basis[i]);
    return res;
}
```

### Problem Set — Bit Manipulation + XOR

| Problem | CF Link | Rating | What to practice |
|---|---|---|---|
| 1554D — Diane | https://codeforces.com/problemset/problem/1554/D | 1700 | XOR construction |
| 1556E — Solving the Knapsack | https://codeforces.com/problemset/problem/1556/E | 1800 | XOR properties |
| 1516E ——tion Deck | https://codeforces.com/problemset/problem/1516/E | 2100 | XOR linear basis |
| 1625E — Bit Arranging | https://codeforces.com/problemset/problem/1625/E | 1700 | Bit manipulation |
| 1832D — More XOR | https://codeforces.com/problemset/problem/1832/D | 1800 | XOR properties |
| 1847D — Weak Divisor | https://codeforces.com/problemset/problem/1847/D | 1700 | Bit + math |
| 1560G — Gregor and Two Painters | https://codeforces.com/problemset/problem/1560/G | 2200 | Bit counting |
| 1055E — Very hard problem | https://codeforces.com/problemset/problem/1055/E | 2100 | XOR trie |
| 1017F — The Supersonic | https://codeforces.com/problemset/problem/1017/F | 2100 | Bitmask DP |
| 1208F — Bits And Pieces | https://codeforces.com/problemset/problem/1208/F | 2400 | Bit trie + dp |

**Searchable tags:**
- https://codeforces.com/problemset?tags=bitmasks&order=BY_RATING_ASC
- https://codeforces.com/problemset?tags=bitmasks,greedy&order=BY_RATING_ASC

---

## 11. Prefix Sums + Difference Arrays

### When to use what
- **Prefix sum:** static array, range sum queries in O(1) after O(n) build
- **2D prefix sum:** range sum on a rectangle
- **Difference array:** apply range additions in O(1), then prefix sum once to get final array
- **Prefix sum + binary search:** "find the smallest index where prefix sum ≥ k"

```cpp
// 1D prefix sum
vector<long long> pre(n + 1, 0);
for (int i = 0; i < n; i++) pre[i + 1] = pre[i] + a[i];
// sum(l, r) inclusive = pre[r+1] - pre[l]

// Difference array for range add
vector<int> diff(n + 1, 0);
// add v to [l, r] inclusive:
diff[l] += v;
diff[r + 1] -= v;
// after all updates:
for (int i = 1; i < n; i++) diff[i] += diff[i - 1];

// 2D prefix sum
int pre2[N][M];
for (int i = 1; i <= n; i++)
    for (int j = 1; j <= m; j++)
        pre2[i][j] = a[i][j] + pre2[i-1][j] + pre2[i][j-1] - pre2[i-1][j-1];
// sum of rectangle (r1,c1) to (r2,c2):
// pre2[r2][c2] - pre2[r1-1][c2] - pre2[r2][c1-1] + pre2[r1-1][c1-1]
```

### Problem Set — Prefix Sums

| Problem | CF Link | Rating | What to practice |
|---|---|---|---|
| 466C — Number of Ways | https://codeforces.com/problemset/problem/466/C | 1700 | Prefix sum counting |
| 1398C — Good Subarrays | https://codeforces.com/problemset/problem/1398/C | 1500 | Prefix sums + counting |
| 1556B — Take a Square | https://codeforces.com/problemset/problem/1556/B | 1400 | 2D prefix sum |
| 1569C — Jury Meeting | https://codeforces.com/problemset/problem/1569/C | 1700 | Prefix + suffix tracking |
| 1547D — Co-prime Counting | https://codeforces.com/problemset/problem/1547/D | 1700 | Prefix sums + math |
| 1364B — Most socially distant | https://codeforces.com/problemset/problem/1364/B | 1300 | Prefix sums |
| 1850F — Brave Mice | https://codeforces.com/problemset/problem/1850/F | 1700 | Diff array |
| 1805D — A and B | https://codeforces.com/problemset/problem/1805/D | 1600 | Prefix max/min |
| 1703D — Double Strings | https://codeforces.com/problemset/problem/1703/D | 1700 | Prefix sums + hashing |
| 1272D — Remove One Element | https://codeforces.com/problemset/problem/1272/D | 1600 | Prefix/suffix DP |

---

## 12. Segment Tree + Fenwick Tree

### When to use
- **BIT (Fenwick):** point updates + prefix sum queries — simpler, faster constant
- **Segment tree:** range queries + range updates (with lazy propagation), more flexible
- **Use BIT first** if it's just "sum/count over prefix" — segment tree is overkill

### BIT (Fenwick Tree)
```cpp
long long bit[MAXN];
int n;

void update(int i, long long val) {
    for (i++; i <= n; i += i & (-i))
        bit[i] += val;
}

long long query(int i) {   // prefix sum [0, i]
    long long s = 0;
    for (i++; i > 0; i -= i & (-i))
        s += bit[i];
    return s;
}

long long query(int l, int r) { return query(r) - (l > 0 ? query(l - 1) : 0); }
```

### Segment Tree (point update, range query)
```cpp
long long tree[4 * MAXN];

void build(long long a[], int node, int l, int r) {
    if (l == r) { tree[node] = a[l]; return; }
    int mid = (l + r) / 2;
    build(a, 2*node, l, mid);
    build(a, 2*node+1, mid+1, r);
    tree[node] = tree[2*node] + tree[2*node+1];  // change for max/min/gcd
}

void update(int node, int l, int r, int pos, long long val) {
    if (l == r) { tree[node] = val; return; }
    int mid = (l + r) / 2;
    if (pos <= mid) update(2*node, l, mid, pos, val);
    else update(2*node+1, mid+1, r, pos, val);
    tree[node] = tree[2*node] + tree[2*node+1];
}

long long query(int node, int l, int r, int ql, int qr) {
    if (qr < l || r < ql) return 0;  // identity for +; use -INF for max
    if (ql <= l && r <= qr) return tree[node];
    int mid = (l + r) / 2;
    return query(2*node, l, mid, ql, qr) + query(2*node+1, mid+1, r, ql, qr);
}
```

### Problem Set — Segment Tree / BIT

**BIT (simpler, practice first):**

| Problem | CF Link | Rating | What to practice |
|---|---|---|---|
| 380C — Sereja and Brackets | https://codeforces.com/problemset/problem/380/C | 2000 | Segment tree merge |
| 1093G — Multidimensional Queries | https://codeforces.com/problemset/problem/1093/G | 2400 | Segment tree |
| 616E — Sum of Remainders | https://codeforces.com/problemset/problem/616/E | 1800 | BIT + math |
| 1619H — Permutation and Queries | https://codeforces.com/problemset/problem/1619/H | 2700 | BIT / sqrt decomp |
| 1477C — Nezzar's Toy | https://codeforces.com/problemset/problem/1477/C | 1600 | BIT counting |
| 1593E — Sakurako and Long Tournament | https://codeforces.com/problemset/problem/1593/E | 1700 | BIT |

**Segment tree:**

| Problem | CF Link | Rating | What to practice |
|---|---|---|---|
| 339D — XOR and Favorite Number | https://codeforces.com/problemset/problem/339/D | 1700 | Segment tree + XOR |
| 1285F — Classical? | https://codeforces.com/problemset/problem/1285/F | 2200 | Segment tree |
| 1558E — Equalize the Arrays | https://codeforces.com/problemset/problem/1558/E | 1800 | Seg tree / sorting |
| 1551F — Equidistant Vertices | https://codeforces.com/problemset/problem/1551/F | 2200 | Tree + seg tree |
| 1701E — Fib-tree | https://codeforces.com/problemset/problem/1701/E | 2300 | Seg tree + recursion |

**Searchable tags:**
- https://codeforces.com/problemset?tags=data+structures&order=BY_RATING_ASC
- https://codeforces.com/problemset?tags=segment+tree&order=BY_RATING_ASC

---

## 13. Two Pointers + Sliding Window

### When to use
- "Longest/shortest subarray satisfying condition X" where X is monotone (adding elements makes it easier or harder)
- "Count pairs (i, j) with condition C(a[i], a[j])" — sort + two pointers often reduces O(n²) to O(n log n)
- Fixed-size window (k): slide and maintain aggregate efficiently

```cpp
// Two pointers: count pairs with a[i] + a[j] <= target
// Sort first
sort(a.begin(), a.end());
int lo = 0, hi = n - 1, count = 0;
while (lo < hi) {
    if (a[lo] + a[hi] <= target) {
        count += hi - lo;  // all (lo, lo+1), (lo, lo+2) ... (lo, hi) work
        lo++;
    } else {
        hi--;
    }
}

// Sliding window: longest subarray with sum <= k
int l = 0;
long long sum = 0, ans = 0;
for (int r = 0; r < n; r++) {
    sum += a[r];
    while (sum > k) sum -= a[l++];
    ans = max(ans, (long long)(r - l + 1));
}
```

### Problem Set — Two Pointers

| Problem | CF Link | Rating | What to practice |
|---|---|---|---|
| 1697C — awoo's Favorite Problem | https://codeforces.com/problemset/problem/1697/C | 1500 | Two pointers on strings |
| 1713D — Tournament Countdown | https://codeforces.com/problemset/problem/1713/D | 1700 | Two pointers |
| 1462E — Close Distance | https://codeforces.com/problemset/problem/1462/E | 1600 | Sliding window |
| 1736D — Match & Catch | https://codeforces.com/problemset/problem/1736/D | 2100 | Sliding window + hashing |
| 1358E — Are You Fired? | https://codeforces.com/problemset/problem/1358/E | 2000 | Two pointers + greedy |
| 1195D — Submarine | https://codeforces.com/problemset/problem/1195/D | 1600 | Two pointers |
| 1237D — Balanced Ternary | https://codeforces.com/problemset/problem/1237/D | 1600 | Two pointer greedy |
| 1601B — Make It Increasing | https://codeforces.com/problemset/problem/1601/B | 1500 | Two pointers |

**Searchable tags:**
- https://codeforces.com/problemset?tags=two+pointers&order=BY_RATING_ASC

---

## 14. Constructive Algorithms

### Recognition
- "Construct any valid arrangement/sequence satisfying..."
- "Does such an object exist? If so, output one."
- Constraints given in a very specific form (degree sequences, exact counts of a pattern, exactly one valid answer)

### Process
1. Solve n = 1, 2, 3, 4 by hand completely
2. Check necessary conditions (parity, degree sum, count feasibility) — output "NO" early
3. Try greedy construction: most-constrained element first, or largest/smallest first
4. If construction exists, prove it by induction (or verify on examples)

### Problem Set — Constructive

| Problem | CF Link | Rating | What to practice |
|---|---|---|---|
| 1733C — Parity Shuffle Sorting | https://codeforces.com/problemset/problem/1733/C | 1500 | Construction |
| 1660F — Koala and Lights | https://codeforces.com/problemset/problem/1660/F | 1900 | Constructive + parity |
| 1695C — Zero Path | https://codeforces.com/problemset/problem/1695/C | 1400 | Constructive |
| 1556F — Sports Betting | https://codeforces.com/problemset/problem/1556/F | 2100 | Constructive + greedy |
| 1547F — Array Stabilization | https://codeforces.com/problemset/problem/1547/F | 2000 | Constructive + binary |
| 1817C — Newsroom | https://codeforces.com/problemset/problem/1817/C | 1700 | Constructive + sorting |
| 1559D — Lena and Ropes | https://codeforces.com/problemset/problem/1559/D | 1700 | Constructive |
| 1391C — Cyclic Permutation | https://codeforces.com/problemset/problem/1391/C | 1500 | Constructive |
| 1360G — A/B Sharing | https://codeforces.com/problemset/problem/1360/G | 2000 | Constructive + greedy |
| 1543D — RPG Heroes | https://codeforces.com/problemset/problem/1543/D | 1600 | Constructive |

**Searchable tag:** https://codeforces.com/problemset?tags=constructive+algorithms&order=BY_RATING_ASC

---

## 15. Ad-hoc + Invariants + Parity

### Recognition
- The statement describes a **process** (objects moving, colliding, merging) and asks for the **final state**
- Final answer seems independent of operation order — there's an invariant
- Small, specific constraints on a secondary parameter exist specifically to make the invariant computable
- "Can you always" / "is it possible" style questions with no obvious DP/graph structure

### Process
1. Simulate for n = 3, 4, 5 by hand
2. Ask: does changing operation order change the answer? If not → find the invariant
3. Try to reduce to counting a simple local pattern (adjacent pairs, runs, parities)
4. Prove it before coding — casework bugs under time pressure are the #1 cause of WA here

### Problem Set — Ad-hoc

| Problem | CF Link | Rating | What to practice |
|---|---|---|---|
| 4A — Watermelon | https://codeforces.com/problemset/problem/4/A | 800 | Parity invariant |
| 231A — Team | https://codeforces.com/problemset/problem/231/A | 1000 | Simple casework |
| 158B — Taxi | https://codeforces.com/problemset/problem/158/B | 1400 | Grouping by remainder mod 4 |
| 1676D — X-Sum | https://codeforces.com/problemset/problem/1676/D | 1500 | Ad-hoc observation |
| 1547C — Pair Programming | https://codeforces.com/problemset/problem/1547/C | 1400 | Ad-hoc simulation |
| 1654C — Alice and the Cake | https://codeforces.com/problemset/problem/1654/C | 1600 | Invariant (sum preserved) |
| 1538E — Funny Markets | https://codeforces.com/problemset/problem/1538/E | 1600 | Invariant finding |
| 1555D — Say No to Palindrome | https://codeforces.com/problemset/problem/1555/D | 1500 | Constructive + parity |
| 1696C — Fishermen | https://codeforces.com/problemset/problem/1696/C | 1700 | Ad-hoc + math |
| 1779C — Least Prefix Sum | https://codeforces.com/problemset/problem/1779/C | 1800 | Invariant + greedy |

---

## 16. Game Theory

### Nim and Grundy numbers
- **Nim:** n piles of stones, last to move wins. Winning iff XOR of all pile sizes ≠ 0
- **Sprague-Grundy theorem:** every impartial game position has a Grundy number (nimber)
  - `G(pos) = mex({G(next_pos) for all moves from pos})`
  - `mex(S)` = minimum excludant = smallest non-negative integer not in S
- Combined games: XOR the individual Grundy numbers; non-zero → first player wins

```cpp
// Compute Grundy numbers for a simple game
// g[i] = Grundy number of state i
vector<int> g(MAXN, -1);
int grundy(int state) {
    if (g[state] != -1) return g[state];
    set<int> reachable;
    for (int move : getMoves(state))
        reachable.insert(grundy(move));
    int mex = 0;
    while (reachable.count(mex)) mex++;
    return g[state] = mex;
}
```

### Problem Set — Game Theory

| Problem | CF Link | Rating | What to practice |
|---|---|---|---|
| 1628D — Not Playing | https://codeforces.com/problemset/problem/1628/D | 1700 | Nim variant |
| 1536C — Diluc and Fischl | https://codeforces.com/problemset/problem/1536/C | 1700 | Game theory |
| 1451E — Bitwise Queries | https://codeforces.com/problemset/problem/1451/E | 2100 | XOR game |
| 768C — Vote | https://codeforces.com/problemset/problem/768/C | 1700 | Greedy game |
| 850C — Arpa and a Game with Mojtaba | https://codeforces.com/problemset/problem/850/C | 1900 | Grundy |
| 1537C — Challenging Cliffs | https://codeforces.com/problemset/problem/1537/C | 1600 | Game + construction |
| 1628E — Rectangles | https://codeforces.com/problemset/problem/1628/E | 2000 | Game + geometry |
| 603C — Lieges of Legendre | https://codeforces.com/problemset/problem/603/C | 2300 | Sprague-Grundy |

**Searchable tag:** https://codeforces.com/problemset?tags=games&order=BY_RATING_ASC

---

## 17. String Algorithms

### KMP — Pattern Matching O(n+m)
Key idea: the LPS (longest proper prefix that is also suffix) table lets you skip re-comparisons on mismatch.
```cpp
vector<int> computeLPS(string& pat) {
    int m = pat.size();
    vector<int> lps(m, 0);
    int len = 0;
    int i = 1;
    while (i < m) {
        if (pat[i] == pat[len]) {
            len++;
            lps[i] = len;
            i++;
        } else {
            if (len != 0) {
                len = lps[len - 1];
            } else {
                lps[i] = 0;
                i++;
            }
        }
    }
    return lps;
}
```

### Z-Function — O(n)
`z[i]` = length of longest substring starting at `i` that matches a prefix of the string.
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
```

### Rolling Hash (polynomial hashing)
```cpp
const long long MOD1 = 1e9 + 7, BASE1 = 131;
vector<long long> h, pw;

void build(string& s) {
    int n = s.size();
    h.resize(n + 1); pw.resize(n + 1);
    h[0] = 0; pw[0] = 1;
    for (int i = 0; i < n; i++) {
        h[i + 1] = (h[i] * BASE1 + s[i]) % MOD1;
        pw[i + 1] = pw[i] * BASE1 % MOD1;
    }
}

long long getHash(int l, int r) {  // [l, r] inclusive
    return (h[r + 1] - h[l] * pw[r - l + 1] % MOD1 + MOD1 * MOD1) % MOD1;
}
```

### Problem Set — Strings

| Problem | CF Link | Rating | What to practice |
|---|---|---|---|
| 126E — Falsehood | https://codeforces.com/problemset/problem/126/E | 1800 | KMP application |
| 432D — Prefixes and Suffixes | https://codeforces.com/problemset/problem/432/D | 1900 | KMP LPS table |
| 1063F — String Journey | https://codeforces.com/problemset/problem/1063/F | 2400 | Z + DP |
| 1250E — The Coronation | https://codeforces.com/problemset/problem/1250/E | 2200 | Hashing + binary search |
| 1721E — Reversible Bracket Sequences | https://codeforces.com/problemset/problem/1721/E | 2100 | String + stack |
| 1497E — Square-Free Division | https://codeforces.com/problemset/problem/1497/E | 2000 | Hashing + DP |
| 1051F — The Shortest Statement | https://codeforces.com/problemset/problem/1051/F | 2000 | LCA + string |
| 1535F — String Distance | https://codeforces.com/problemset/problem/1535/F | 2300 | Hashing |

**Searchable tags:**
- https://codeforces.com/problemset?tags=strings&order=BY_RATING_ASC
- https://codeforces.com/problemset?tags=string+suffix+structures&order=BY_RATING_ASC

---

## 18. Practice Strategy

### What to do in the weeks before

**Weekly schedule:**
- 2 Codeforces Div 2 contests (or Div 1+2) — live, not upsolve
- 1 virtual contest from actual ICPC India archives (see links below)
- 2–3 hours of targeted upsolve on your weak areas using the problem sets above

**ICPC India archive for virtual practice:**
- 2025–26: https://codeforces.com/gym/106179
- Full archive 2008–2023: https://codeforces.com/blog/entry/105000
- ICPC India problem list: https://github.com/m-e-r-l-i-n/icpc-india

### How to upsolve (do this, not "read editorial and move on")
1. **Time-box first attempt** — 30 min of genuine effort
2. **Look at only the tag** — get the category, not the solution, then try again
3. **If still stuck** — read the first sentence of editorial, close it, implement from scratch
4. **Never copy-paste** — re-deriving from scratch is what builds recognition
5. **Write one sentence after each problem:** "I should have seen this as [pattern] because the statement said [X]"

### Templates to write from memory (practice this regularly)
```
Must be instant:
[ ] DSU with union by rank + path compression
[ ] Weighted/parity DSU
[ ] Dijkstra with priority queue
[ ] 0-1 BFS with deque
[ ] BFS on augmented state
[ ] Binary search on answer skeleton
[ ] nCr mod p (precompute factorials + inverse factorials)
[ ] Sieve of Eratosthenes
[ ] Binary lifting LCA
[ ] BIT (Fenwick tree) — update + query
[ ] Segment tree — basic point update + range query
[ ] KMP LPS table + search
[ ] Z-function
```

### Final week checklist
- [ ] Re-read the recognition checklist (Section 2) daily
- [ ] Do 2–3 past ICPC India sets under real time pressure (2.5h, no lookups)
- [ ] Simulate ICPC reading order: read all 6 statements first, classify by difficulty, solve easiest first
- [ ] Decide in advance: "if I have no key idea after 25 minutes, I switch to the next unread problem"
- [ ] Confirm language/compiler rules for current year on official page
- [ ] Test your template file compiles cleanly under G++17 with `-O2`

---

## 19. Quick Reference

| If the statement says... | Try... |
|---|---|
| "Same group / merge / connected / must be equal" (pairwise) | DSU (plain or weighted) |
| "XOR / difference / sum of two values is fixed" (pairwise) | Weighted DSU |
| "Minimum moves/steps" on graph or grid | BFS (0-1 BFS if 0/1 weights, Dijkstra otherwise) |
| Grid + keys/fuel/parity that affects future moves | BFS over (position, extra state) |
| "Rearrange to minimize/maximize" | Sort first, then greedy + exchange argument |
| "Largest/smallest X such that condition holds" (condition monotone) | Binary search on X |
| "Number of ways... mod 1e9+7" | Counting DP or combinatorics with modular inverse |
| "Count pairs/subsets with GCD = k" | Fix-the-GCD sieve trick |
| "XOR / binary representation / bits" | Per-bit decomposition; trie for max XOR |
| n ≤ 20, "subset" | Bitmask DP / brute subsets |
| "Construct any valid arrangement" | Small cases by hand → necessary conditions → greedy order |
| Process described, final answer seems order-independent | Find the invariant, reduce to counting local pattern |
| Tree given explicitly | Root it; subtree DP or rerooting; LCA for path queries |
| "Maximum XOR of two elements in array" | Bit trie |
| Range sum queries (static array) | Prefix sums |
| Range queries + point updates | BIT (Fenwick tree) |
| Range queries + range updates | Segment tree with lazy propagation |
| "Longest/shortest subarray with property X" (monotone) | Two pointers / sliding window |
| Last player to move wins, take from piles | Nim — XOR of pile sizes |
| Linear recurrence, n up to 10^18 | Matrix exponentiation |

---

> **Most important reminder:** The skill that separates rank 5 from rank 200 at ICPC India prelims is **not knowing more algorithms**. Look at *Collisions*, *Equations*, *Pseudo Palindrome*, *Small Indices* — none of them are "implement segment tree." They are: model the situation correctly, find the right invariant, implement carefully under time pressure. Drill the recognition reflex on real problems, not algorithm theory.