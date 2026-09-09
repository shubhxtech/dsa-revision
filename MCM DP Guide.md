# Interval DP / MCM — Full Deep-Dive Guide (C++)

This is a standalone companion to the Graphs+DP roadmap, expanding just the interval DP / Matrix
Chain Multiplication pattern into a complete guide: how to recognize it, how to derive the
recurrence yourself under pressure, full C++ for every famous problem, and the mistakes that
usually cause TLE/WA on this specific pattern.

---

## 1. What interval DP actually is

Almost every other DP pattern you've done (knapsack, LIS, grid DP) has a state that grows in one
direction — you add one element at a time and never look back. Interval DP is different: the state
is a **range `[l, r]`**, and you solve it by **combining the answers of two or more sub-ranges
inside it**. That's it. That's the whole idea. The reason it looks intimidating is that the "last
operation" that splits `[l, r]` isn't always an obvious cut — sometimes it's a matched bracket
pair, sometimes it's "which element do I remove last," sometimes it's a two-player move from either
end.

**Why Matrix Chain Multiplication is the archetype:** in MCM you're deciding the order to multiply
`A_1 x A_2 x ... x A_n`. You can't be greedy about it, because the cost of combining two chains
depends on their outer dimensions — but here's the key fact that makes the whole pattern work:

> **The result of fully multiplying any sub-chain `[l, r]` has fixed dimensions, no matter how you
> internally parenthesized it.** Only the *cost* to get there changes.

That's what lets you write `dp[l][r]` as a clean function of just `l` and `r`, and try every split
point `k` independently. Every problem in this guide is a variation on that one fact.

---

## 2. How to recognize an interval DP problem

Ask these questions. If the answer to the first is yes and at least one of the others is yes, it's
interval DP:

- **Is there a contiguous sequence (array/string) that gets fully consumed, merged, partitioned, or built into a structure?**
- "Combine adjacent piles/matrices/numbers, minimize/maximize total cost" → yes (MCM, Slimes)
- "Remove elements from a sequence (from the ends, or anywhere with a rule) until it's empty, optimize score" → yes (Removal Game, Zuma, Strange Printer, Remove Boxes)
- "Fully parenthesize an expression / count valid ways to bracket something" → yes (Boolean Parenthesization, MCM)
- "Build a binary tree over a sorted/ordered sequence, minimize some tree-shaped cost" → yes (Optimal BST, Min Cost Tree from Leaf Values, Burst Balloons in disguise)
- "Two players alternately take from either end of an array" → yes (Predict the Winner, Removal Game)
- "Is this substring a palindrome / can I partition into palindromes" → yes, often paired with a 1D DP on top (Palindrome Partitioning II)

If you see these words in a problem — **merge, burst, remove, partition, parenthesize, build a
tree from a sequence, take-from-ends game** — go straight to interval DP as your first hypothesis.

---

## 3. The universal framework (do this every time, in order)

1. **Define `dp[l][r]` in one sentence.** E.g. "min cost to merge slimes l..r into one," or "max
   score difference the current player can force on subarray a[l..r]."
2. **Base case.** Always the smallest range: `dp[l][l]` (single element) or `dp[l][l+1]` (a pair),
   depending on the problem.
3. **Decide what the "last operation" is.** This is the only genuinely hard step, and it's
   *always* one of: (a) a split point `k` breaking `[l,r]` into `[l,k]` and `[k+1,r]`, (b) "which
   element gets removed/placed last" (often the *opposite* of what you'd naively guess — see Burst
   Balloons), or (c) a move from the left end vs. the right end (two-player games).
4. **Write the transition covering every choice of the last operation**, taking min/max/sum over
   all of them.
5. **Check if the range alone is enough state.** If the answer also depends on something like "is
   this bracket colored," "how many duplicates are attached," or "which color is at the boundary,"
   add that as extra dimensions: `dp[l][r][extra]`.
6. **Order of computation:** always process ranges by **increasing length**. `dp[l][r]` depends on
   strictly shorter ranges, never on anything of the same or greater length. (See Pitfall #1 below
   — this trips people up more than anything else.)
7. **Complexity gut-check:** `O(n^2)` ranges × `O(n)` choices of `k` = **`O(n^3)`** baseline. Fine
   up to `n ~ 500`. If `n` is larger, you either need a smarter cost formula (prefix sums to make
   `mergeCost` O(1)) or an optimization like Knuth's (Section 8).

---

## 4. The generic templates

### Top-down (write this first — it enforces correct ordering automatically)

This is the version I'd recommend writing first in an interview or OA: recursion handles the
"process shorter ranges first" requirement for you, so you can't make Pitfall #1 below.

```cpp
long long memo[MAXN][MAXN];
bool seen[MAXN][MAXN];

long long solve(int l, int r) {
    if (l == r) return baseCase(l);              // adapt base case to the problem
    if (seen[l][r]) return memo[l][r];
    seen[l][r] = true;

    long long best = INF;                         // or -INF for maximization
    for (int k = l; k < r; k++) {
        long long cand = solve(l, k) + solve(k + 1, r) + mergeCost(l, k, r);
        best = min(best, cand);                    // or max
    }
    return memo[l][r] = best;
}
```

### Bottom-up (convert to this for CP submissions once the logic is verified)

```cpp
int n;
long long dp[MAXN][MAXN];

for (int i = 0; i < n; i++) dp[i][i] = baseCase(i);

for (int len = 2; len <= n; len++) {
    for (int l = 0; l + len - 1 < n; l++) {
        int r = l + len - 1;
        dp[l][r] = INF;                            // or -INF
        for (int k = l; k < r; k++) {
            long long cand = dp[l][k] + dp[k + 1][r] + mergeCost(l, k, r);
            dp[l][r] = min(dp[l][r], cand);
        }
    }
}
// answer is dp[0][n-1]
```

The only three things that ever change between problems: **what `mergeCost` is**, **what the split
point represents**, and **whether you need extra state beyond `[l][r]`**.

---

## 5. Common state-augmentation patterns

| Situation | What to add | Example |
|---|---|---|
| Plain "combine two halves, cost depends only on the range" | Nothing — plain `dp[l][r]` | MCM, Slimes |
| Two-player optimal game, alternating moves from either end | `dp[l][r]` = **score difference**, not raw score | Removal Game, Predict the Winner |
| Counting ways instead of min/max | `dp[l][r][true/false]` or just sum instead of min | Boolean Parenthesization |
| The boundary's color/state affects merging with neighbors | `dp[l][r][boundaryState]` | Coloring Brackets |
| "Last removed" isn't the outer boundary but something reframed | Redefine what `l, r` mean (inclusive vs. exclusive boundary) | Burst Balloons |
| Duplicates of the same value pile up and should burst together | extra count dimension `dp[l][r][k]` | Remove Boxes |
| Need a yes/no over sub-ranges before the main DP | Precompute a boolean table first (e.g. `isPalindrome[l][r]`) | Palindrome Partitioning II, Zuma |

---

## 6. Pitfalls that actually cause bugs

1. **Wrong loop order.** If you fix `l` as the outer loop and increase `r` inside it, `dp[k+1][r]`
   for `k+1 > l` may not be computed yet. **Always loop by length first**, or use the top-down
   version, which sidesteps this entirely.
2. **Off-by-one on the length loop bound.** The inner condition should be `l + len - 1 < n` (0-indexed)
   — a common source of segfaults/wrong answers.
3. **Forgetting the empty-range base case.** Problems like Optimal BST and Burst Balloons need
   `cost(i, j) = 0` when the range is empty (`i > j`), and this has to be handled explicitly, not
   silently assumed to be zero from an uninitialized array.
4. **Integer overflow.** `mergeCost` in MCM-style problems is often a product of large dimensions —
   use `long long`, always.
5. **Forgetting to precompute range sums/prefix data.** If `mergeCost(l, k, r)` itself requires an
   `O(n)` loop to compute a sum, your total complexity silently becomes `O(n^4)` — precompute
   prefix sums so it's `O(1)`.
6. **Reframing errors in "last operation" problems.** Burst Balloons is the classic trap: the
   natural instinct is "which balloon do I burst *first*," but the DP only works cleanly if you
   think about which one you burst *last* in a given range. When your recurrence doesn't seem to
   close nicely, try flipping first vs. last.

---

## 7. Worked problems, in ladder order

Each one: one-line intuition → recurrence → full C++.

### Tier 0 — the pattern in pure form

#### 7.1 Matrix Chain Multiplication (the original)
**Intuition:** `dp[i][j]` = min scalar multiplications to fully multiply matrices `i..j`. Try every
split point `k` for "which multiplication happens last."

`dp[i][j] = min over k in [i, j-1] of dp[i][k] + dp[k+1][j] + p[i-1]*p[k]*p[j]`

```cpp
long long matrixChainOrder(vector<int>& p) { // p has n+1 entries for n matrices
    int n = p.size() - 1;
    vector<vector<long long>> dp(n + 1, vector<long long>(n + 1, 0));

    for (int len = 2; len <= n; len++) {
        for (int i = 1; i + len - 1 <= n; i++) {
            int j = i + len - 1;
            dp[i][j] = LLONG_MAX;
            for (int k = i; k < j; k++) {
                long long cost = dp[i][k] + dp[k + 1][j] + (long long)p[i - 1] * p[k] * p[j];
                dp[i][j] = min(dp[i][j], cost);
            }
        }
    }
    return dp[1][n];
}
```

#### 7.2 Boolean Parenthesization (count ways to get `true`)
**Intuition:** same split-point idea, but instead of one cost you track two counts per range: ways
to make it `True` and ways to make it `False`, combined according to the operator at the split.

```cpp
long long countWays(string symbols, string operators) {
    int n = symbols.size();
    vector<vector<long long>> T(n, vector<long long>(n, 0)), F(n, vector<long long>(n, 0));

    for (int i = 0; i < n; i++) {
        T[i][i] = (symbols[i] == 'T');
        F[i][i] = (symbols[i] == 'F');
    }

    for (int len = 2; len <= n; len++) {
        for (int i = 0; i + len - 1 < n; i++) {
            int j = i + len - 1;
            for (int k = i; k < j; k++) {
                char op = operators[k]; // operator between symbol k and k+1
                long long totalL = T[i][k] + F[i][k];
                long long totalR = T[k + 1][j] + F[k + 1][j];
                long long tWays = 0, fWays = 0;

                if (op == '&') {
                    tWays = T[i][k] * T[k + 1][j];
                    fWays = totalL * totalR - tWays;
                } else if (op == '|') {
                    fWays = F[i][k] * F[k + 1][j];
                    tWays = totalL * totalR - fWays;
                } else { // '^'
                    tWays = T[i][k] * F[k + 1][j] + F[i][k] * T[k + 1][j];
                    fWays = totalL * totalR - tWays;
                }
                T[i][j] += tWays;
                F[i][j] += fWays;
            }
        }
    }
    return T[0][n - 1];
}
```

#### 7.3 Palindrome Partitioning II (LeetCode 132)
**Intuition:** first, interval DP builds an `isPal[l][r]` table. Then a *separate*, simpler 1D DP
finds the minimum cuts using that table — a good bridge problem between interval DP and plain 1D DP.

```cpp
int minCut(string s) {
    int n = s.size();
    vector<vector<bool>> isPal(n, vector<bool>(n, false));
    for (int i = 0; i < n; i++) isPal[i][i] = true;

    for (int len = 2; len <= n; len++) {
        for (int i = 0; i + len - 1 < n; i++) {
            int j = i + len - 1;
            if (s[i] == s[j] && (len == 2 || isPal[i + 1][j - 1]))
                isPal[i][j] = true;
        }
    }

    vector<int> cut(n, INT_MAX);
    for (int i = 0; i < n; i++) {
        if (isPal[0][i]) { cut[i] = 0; continue; }
        for (int j = 0; j < i; j++) {
            if (isPal[j + 1][i] && cut[j] != INT_MAX)
                cut[i] = min(cut[i], cut[j] + 1);
        }
    }
    return cut[n - 1];
}
```

---

### Tier 1 — the direct MCM twins

#### 7.4 AtCoder DP Contest N — Slimes
**Intuition:** literally MCM with `mergeCost = sum(l, r)` instead of a dimension product. Do this
one first among the "twins" — it's the cleanest possible restatement of the template.

```cpp
long long minSlimeCost(vector<long long>& a) {
    int n = a.size();
    vector<long long> prefix(n + 1, 0);
    for (int i = 0; i < n; i++) prefix[i + 1] = prefix[i] + a[i];

    vector<vector<long long>> dp(n, vector<long long>(n, 0));
    for (int len = 2; len <= n; len++) {
        for (int l = 0; l + len - 1 < n; l++) {
            int r = l + len - 1;
            long long rangeSum = prefix[r + 1] - prefix[l];
            dp[l][r] = LLONG_MAX;
            for (int k = l; k < r; k++)
                dp[l][r] = min(dp[l][r], dp[l][k] + dp[k + 1][r] + rangeSum);
        }
    }
    return dp[0][n - 1];
}
```

#### 7.5 CSES 1097 — Removal Game
**Intuition:** the twist for two-player games — `dp[l][r]` is not "my max score," it's **the best
score difference (me minus opponent) achievable on `a[l..r]`**, because whichever player moves
next on a sub-range is whoever didn't move last, and the difference framing makes both players'
optimal play fall out of the same recurrence.

`dp[l][r] = max(a[l] - dp[l+1][r], a[r] - dp[l][r-1])`

```cpp
long long removalGame(vector<long long>& a) {
    int n = a.size();
    vector<vector<long long>> dp(n, vector<long long>(n, 0));
    for (int i = 0; i < n; i++) dp[i][i] = a[i];

    for (int len = 2; len <= n; len++) {
        for (int l = 0; l + len - 1 < n; l++) {
            int r = l + len - 1;
            dp[l][r] = max(a[l] - dp[l + 1][r], a[r] - dp[l][r - 1]);
        }
    }
    long long total = accumulate(a.begin(), a.end(), 0LL);
    return (total + dp[0][n - 1]) / 2; // first player's actual total score
}
```

#### 7.6 LeetCode 486 — Predict the Winner
**Intuition:** identical recurrence to Removal Game — solve it and check the sign of `dp[0][n-1]`
to confirm you can transfer the pattern without re-deriving from scratch.

```cpp
bool PredictTheWinner(vector<int>& nums) {
    int n = nums.size();
    vector<vector<int>> dp(n, vector<int>(n, 0));
    for (int i = 0; i < n; i++) dp[i][i] = nums[i];

    for (int len = 2; len <= n; len++) {
        for (int l = 0; l + len - 1 < n; l++) {
            int r = l + len - 1;
            dp[l][r] = max(nums[l] - dp[l + 1][r], nums[r] - dp[l][r - 1]);
        }
    }
    return dp[0][n - 1] >= 0;
}
```

#### 7.7 LeetCode 1130 — Minimum Cost Tree From Leaf Values
**Intuition:** back to the "MCM shape" — `mergeCost` is now `max(l..k) * max(k+1..r)`. Precompute
range-max first, then it's the plain template.

```cpp
int mctFromLeafValues(vector<int>& arr) {
    int n = arr.size();
    vector<vector<int>> mx(n, vector<int>(n, 0));
    for (int i = 0; i < n; i++) mx[i][i] = arr[i];
    for (int len = 2; len <= n; len++)
        for (int l = 0; l + len - 1 < n; l++) {
            int r = l + len - 1;
            mx[l][r] = max(mx[l][r - 1], arr[r]);
        }

    vector<vector<long long>> dp(n, vector<long long>(n, 0));
    for (int len = 2; len <= n; len++) {
        for (int l = 0; l + len - 1 < n; l++) {
            int r = l + len - 1;
            dp[l][r] = LLONG_MAX;
            for (int k = l; k < r; k++)
                dp[l][r] = min(dp[l][r], dp[l][k] + dp[k + 1][r] + (long long)mx[l][k] * mx[k + 1][r]);
        }
    }
    return (int)dp[0][n - 1];
}
```

---

### Tier 2 — the split point stops being obvious

#### 7.8 LeetCode 312 — Burst Balloons
**Intuition — the big "aha":** don't think about which balloon you burst first. Think about which
balloon in range `(l, r)` (exclusive boundaries) you burst **last**. Everything to its left and
everything to its right gets resolved independently, and when you finally burst it, its only
remaining neighbors are `nums[l]` and `nums[r]` (the two boundary balloons, which by definition
haven't been popped yet). Pad the array with `1` on both ends so boundaries always exist.

```cpp
int maxCoins(vector<int>& nums) {
    int n = nums.size();
    vector<int> a(n + 2);
    a[0] = a[n + 1] = 1;
    for (int i = 0; i < n; i++) a[i + 1] = nums[i];

    vector<vector<int>> dp(n + 2, vector<int>(n + 2, 0));
    for (int len = 2; len <= n + 1; len++) {
        for (int l = 0; l + len <= n + 1; l++) {
            int r = l + len;
            for (int k = l + 1; k < r; k++) {
                dp[l][r] = max(dp[l][r], dp[l][k] + dp[k][r] + a[l] * a[k] * a[r]);
            }
        }
    }
    return dp[0][n + 1];
}
```

#### 7.9 Optimal Binary Search Tree
**Intuition:** MCM shape again, but the split point `r` (the chosen root) doesn't just divide into
two independent halves being multiplied — it adds `sum(freq[i..j])` once per level of depth, which
telescopes out to "add the range sum once per recursive call," same bookkeeping trick as Slimes.

```cpp
int optimalBST(vector<int>& keys, vector<int>& freq) {
    int n = keys.size();
    vector<int> prefix(n + 1, 0);
    for (int i = 0; i < n; i++) prefix[i + 1] = prefix[i] + freq[i];
    auto rangeSum = [&](int i, int j) { return prefix[j + 1] - prefix[i]; };

    vector<vector<int>> cost(n, vector<int>(n, 0));
    for (int i = 0; i < n; i++) cost[i][i] = freq[i];

    for (int len = 2; len <= n; len++) {
        for (int i = 0; i + len - 1 < n; i++) {
            int j = i + len - 1;
            cost[i][j] = INT_MAX;
            int sum = rangeSum(i, j);
            for (int r = i; r <= j; r++) {
                int left = (r > i) ? cost[i][r - 1] : 0;
                int right = (r < j) ? cost[r + 1][j] : 0;
                cost[i][j] = min(cost[i][j], left + right + sum);
            }
        }
    }
    return cost[0][n - 1];
}
```

#### 7.10 Codeforces 149D — Coloring Brackets (extra-state example, ~2000)
**Intuition (sketch — verify exact adjacency rules against the statement before submitting):** this
is the canonical example of needing state beyond `[l][r]`. A well-formed bracket sequence naturally
decomposes into matched pairs, so `match[i]` gives you the partner of each bracket. Define
`dp[l][r][cl][cr]` = number of valid colorings of `s[l..r]` where the bracket at `l` uses color `cl`
and the bracket at `r` uses color `cr` (colors: none / red / blue). The transition combines: (a) the
matched pair `(l, match[l])`, which must not be both the same color and not both uncolored, (b) the
interior `(l+1, match[l]-1)` recursively, and (c) whatever follows after `match[l]` up to `r`,
respecting the "no two adjacent same-colored brackets" rule at the junction. This one is genuinely
fiddly — treat it as proof that the state-augmentation idea generalizes, not as a template to copy
verbatim; re-derive the exact color-compatibility table from the problem statement when you solve it.

```cpp
// Skeleton only — fill in exact color-compatibility checks from the problem statement.
// dp[l][r][cl][cr]: colors are 0=none, 1=red, 2=blue
long long dp[MAXN][MAXN][3][3];
int match_[MAXN];

// base: dp[i][i+1][cl][cr] for a directly matched pair (l, l+1)
// recursive: combine (l, match[l]) with interior, then attach the remainder (match[l]+1 .. r)
// checking adjacency compatibility at every junction of colors.
```

---

### Tier 3 — combined with another structure

#### 7.11 Codeforces 607B — Zuma (~1900)
**Intuition:** you can erase any contiguous palindromic segment in one move. `dp[l][r]` = min moves
to clear `a[l..r]`. If the two ends match, you can "wrap" the solution for the interior in one
extra pass; otherwise fall back to the general split.

```cpp
int minZumaMoves(vector<int>& a) {
    int n = a.size();
    vector<vector<int>> dp(n, vector<int>(n, 0));
    for (int i = 0; i < n; i++) dp[i][i] = 1;

    for (int len = 2; len <= n; len++) {
        for (int l = 0; l + len - 1 < n; l++) {
            int r = l + len - 1;
            if (len == 2) {
                dp[l][r] = (a[l] == a[r]) ? 1 : 2;
                continue;
            }
            dp[l][r] = INT_MAX;
            if (a[l] == a[r]) dp[l][r] = dp[l + 1][r - 1];
            for (int k = l; k < r; k++)
                dp[l][r] = min(dp[l][r], dp[l][k] + dp[k + 1][r]);
        }
    }
    return dp[0][n - 1];
}
```

#### 7.12 Codeforces 1025D — Recovering BST (extra-state example, ~2000)
**Intuition (sketch):** you need a BST over a sorted sequence `a[0..n-1]` where every parent-child
edge has `gcd > 1`. The trick: for a sub-range `[l, r]` you don't need to know the *entire* subtree
shape — only whether it's possible to build a valid subtree over `[l, r]` **whose root is the
leftmost element `l`** (so it can attach to a parent on its right) or **whose root is the rightmost
element `r`** (so it can attach to a parent on its left). Track two boolean tables, `canLeft[l][r]`
and `canRight[l][r]`, and transition by trying every position `i` in `[l, r]` as the root, checking
`gcd(a[i], parent) > 1` against the appropriate boundary table of the two resulting sub-ranges.
As with 149D, re-derive the exact transition carefully from the statement — the two-boolean-table
trick is the reusable idea here, more than the literal code.

```cpp
// Skeleton — canLeft[l][r] = can [l,r] form a valid subtree rooted at position l
//            canRight[l][r] = can [l,r] form a valid subtree rooted at position r
bool canLeft[MAXN][MAXN], canRight[MAXN][MAXN];
// base case: canLeft[i][i] = canRight[i][i] = true (single node)
// transition: try every root position i in [l, r], require gcd compatibility with
// the boundary of the left part (canRight[l][i-1]) and right part (canLeft[i+1][r])
```

#### 7.13 LeetCode 664 — Strange Printer
**Intuition:** default cost is "print the last character separately" (`dp[l][r-1] + 1`), but
whenever an earlier character in the range matches `s[r]`, you can print them in the *same* pass,
merging the interior cost for free.

```cpp
int strangePrinter(string s) {
    int n = s.size();
    vector<vector<int>> dp(n, vector<int>(n, 0));
    for (int i = n - 1; i >= 0; i--) {
        dp[i][i] = 1;
        for (int j = i + 1; j < n; j++) {
            dp[i][j] = dp[i][j - 1] + 1;
            for (int k = i; k < j; k++) {
                if (s[k] == s[j]) {
                    int mid = (k + 1 <= j - 1) ? dp[k + 1][j - 1] : 0;
                    dp[i][j] = min(dp[i][j], dp[i][k] + mid);
                }
            }
        }
    }
    return dp[0][n - 1];
}
```

---

### Tier 4 — optional stretch (past OA level, good for ICPC)

#### 7.14 LeetCode 546 — Remove Boxes (the hardest common interval DP problem)
**Intuition:** the range alone isn't enough — you also need to remember "how many boxes of the same
color as `boxes[r]` are sitting just outside `r`, waiting to be burst together for a bonus." That's
the third dimension `k`. Removing `k+1` same-colored boxes together scores `(k+1)^2`, which is why
it's sometimes worth *not* bursting them immediately and instead attaching more matches first.

```cpp
class RemoveBoxesSolver {
public:
    vector<int> boxes;
    int memo[100][100][100];

    int calculatePoints(vector<int>& b) {
        boxes = b;
        memset(memo, -1, sizeof(memo));
        return solve(0, (int)boxes.size() - 1, 0);
    }

    int solve(int l, int r, int k) {
        if (l > r) return 0;
        if (memo[l][r][k] != -1) return memo[l][r][k];

        int rr = r, kk = k;
        while (rr > l && boxes[rr] == boxes[rr - 1]) { rr--; kk++; }

        int res = (kk + 1) * (kk + 1) + solve(l, rr - 1, 0);
        for (int m = l; m < rr; m++) {
            if (boxes[m] == boxes[rr]) {
                res = max(res, solve(l, m, kk + 1) + solve(m + 1, rr - 1, 0));
            }
        }
        return memo[l][r][k] = res;
    }
};
```

---

## 8. Bonus: Knuth's optimization (only after Tiers 0–3 feel easy)

Some interval DP problems (including classic MCM) satisfy a property that lets you speed up from
`O(n^3)` to `O(n^2)`: the optimal split point `opt[l][r]` is monotonic —
`opt[l][r-1] <= opt[l][r] <= opt[l+1][r]`. Instead of trying every `k` in `[l, r)`, you only try `k`
in `[opt[l][r-1], opt[l+1][r]]`.

```cpp
int opt[MAXN][MAXN];
for (int i = 0; i < n; i++) { dp[i][i] = baseCase(i); opt[i][i] = i; }

for (int len = 2; len <= n; len++) {
    for (int l = 0; l + len - 1 < n; l++) {
        int r = l + len - 1;
        int lo = (l < r) ? opt[l][r - 1] : l;
        int hi = (l < r) ? opt[l + 1][r] : r;
        dp[l][r] = INF;
        for (int k = lo; k <= hi; k++) {
            long long cand = dp[l][k] + dp[k + 1][r] + mergeCost(l, k, r);
            if (cand < dp[l][r]) { dp[l][r] = cand; opt[l][r] = k; }
        }
    }
}
```

This is an ICPC-level nicety, not something OAs will require — only reach for it if `n` is too
large for `O(n^3)` and the cost function looks like it should satisfy the quadrangle inequality
(most "sum of a range" or "product of prefix sums" cost functions do).

---

## 9. Recap: the ladder

1. Matrix Chain Multiplication → 2. Boolean Parenthesization → 3. Palindrome Partitioning II
→ 4. AtCoder DP N (Slimes) → 5. CSES Removal Game → 6. Predict the Winner → 7. Min Cost Tree From Leaf Values
→ 8. Burst Balloons → 9. Optimal BST → 10. Coloring Brackets
→ 11. Zuma → 12. Recovering BST → 13. Strange Printer
→ 14. Remove Boxes (stretch) → Knuth's optimization (bonus, conceptual)

If you only have time for a subset: **1 → 4 → 5 → 8** covers the plain template, the game-theory
variant, and the "reframe the last operation" trap — that combination alone will make most OA/ICPC
interval DP problems recognizable on sight.