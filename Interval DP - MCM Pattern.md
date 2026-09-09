# Interval DP — MCM Pattern (Deep Dive)

> **One template. Infinite disguises.**
> Matrix Chain Multiplication, Burst Balloons, Remove Boxes — they all look different. They're not.
> Once you see the pattern, you'll recognize it in 30 seconds.

---

## Table of Contents

| # | Topic |
|---|-------|
| 1 | [What is Interval DP?](#1-what-is-interval-dp) |
| 2 | [How to Recognize It](#2-how-to-recognize-it) |
| 3 | [The Universal Framework](#3-the-universal-framework) |
| 4 | [Generic Template — Code](#4-generic-template--code) |
| 5 | [Problem 1 — Matrix Chain Multiplication](#5-problem-1--matrix-chain-multiplication) |
| 6 | [Problem 2 — Boolean Parenthesization](#6-problem-2--boolean-parenthesization) |
| 7 | [Problem 3 — Palindrome Partitioning II](#7-problem-3--palindrome-partitioning-ii-lc-132) |
| 8 | [Problem 4 — Slimes (AtCoder DP N)](#8-problem-4--slimes-atcoder-dp-n) |
| 9 | [Problem 5 — Removal Game / Predict the Winner](#9-problem-5--removal-game--predict-the-winner) |
| 10 | [Problem 6 — Burst Balloons (LC 312)](#10-problem-6--burst-balloons-lc-312) |
| 11 | [Problem 7 — Min Cost Tree from Leaf Values (LC 1130)](#11-problem-7--min-cost-tree-from-leaf-values-lc-1130) |
| 12 | [Problem 8 — Strange Printer (LC 664)](#12-problem-8--strange-printer-lc-664) |
| 13 | [Problem 9 — Optimal BST](#13-problem-9--optimal-bst) |
| 14 | [Problem 10 — Remove Boxes (LC 546)](#14-problem-10--remove-boxes-lc-546-hardest) |
| 15 | [Common Pitfalls](#15-common-pitfalls) |
| 16 | [Knuth's Optimization (Bonus)](#16-knuths-optimization-bonus) |
| 17 | [Practice Ladder](#17-practice-ladder) |

---

## 1. What is Interval DP?

Most DP patterns you know work **left to right** — you process elements one by one and never look back (knapsack, LIS, grid DP).

**Interval DP is different.** The state is a **range `[l, r]`** over a sequence, and you solve it by **combining answers for sub-ranges inside it**.

### The core idea in one picture:

```
We want to solve [l ............. r]

We split at some point k:
  [l ....... k]  +  [k+1 ....... r]
       ^                  ^
  already solved      already solved

  Then add the "cost of combining" the two halves.
  Try every possible k, take the best.
```

### Why does Matrix Chain Multiplication become the archetype?

When you multiply matrices `A1 x A2 x ... x An`, you can parenthesize in many ways. Each way has a different cost. **The key fact that unlocks the DP:**

> No matter how you internally parenthesize sub-chain `[l, r]`, the result matrix has the
> **same dimensions** (`rows[l] x cols[r]`). Only the *cost* to get there changes.

This means `dp[l][r]` is a clean function of just `l` and `r`. Every problem in this guide is built on this same insight.

---

## 2. How to Recognize It

**Ask these questions. If the first is YES + at least one other — it's Interval DP.**

### Primary signal:
> Is there a contiguous sequence (array or string) that gets **fully merged, consumed, partitioned, or built** into something?

### Secondary signals:

| Problem sounds like... | Classic variant |
|---|---|
| "Combine adjacent piles/matrices, minimize/maximize total cost" | MCM, Slimes |
| "Remove elements (from ends or anywhere with a rule) until empty" | Removal Game, Burst Balloons, Zuma |
| "Fully parenthesize an expression / count valid bracketings" | Boolean Parenthesization, MCM |
| "Build a binary tree over an ordered sequence, optimize tree cost" | Optimal BST, Leaf Values |
| "Two players alternately take from either end" | Predict the Winner, Removal Game |
| "Palindrome partitioning" | Palindrome Partitioning II |

### Trigger words:
**merge, burst, remove, partition, parenthesize, take from ends, build a tree from a sequence** -> think Interval DP first.

---

## 3. The Universal Framework

**Do this in order, every time, before writing any code.**

### Step 1 — Define `dp[l][r]` in one sentence.
Be specific. Examples:
- "Min cost to multiply matrices `l` through `r`"
- "Max coins I can collect from balloons in range `(l, r)` exclusive"
- "Best score difference (me minus opponent) on subarray `a[l..r]`"

### Step 2 — Identify the base case.
Always the smallest valid range:
- `dp[i][i]` — single element (most problems)
- `dp[i][i+1]` — a pair (when single element is trivial)
- Sometimes `dp[i][j] = 0` for `i > j` (empty range)

### Step 3 — Find the "last operation" (the hardest step).
This is always one of:
- **A split point `k`** — which merge happens last? Breaks `[l,r]` into `[l,k]` and `[k+1,r]`
- **Which element is removed/placed last** — NOT first. This is the Burst Balloons trick.
- **A move from the left vs. right end** — two-player game problems.

### Step 4 — Write the transition.
Try every valid choice of the "last operation". Take min/max/sum over all of them.

### Step 5 — Check if `[l][r]` is enough state.
Sometimes you need extra dimensions:
- Two-player game -> `dp[l][r]` = score *difference*, not raw score
- Color/state at boundary matters -> `dp[l][r][color]`
- "Last removed" depends on context outside the range -> add `k` dimension

### Step 6 — Order of computation.
**ALWAYS iterate by increasing length.** `dp[l][r]` uses shorter sub-ranges, never same or longer.
```cpp
for (int len = 2; len <= n; len++)       // outer loop: length
    for (int l = 0; l + len - 1 < n; l++)
        // compute dp[l][r] where r = l + len - 1
```

### Step 7 — Complexity check.
`O(n^2)` ranges * `O(n)` split points = **O(n^3)**. Fine for `n <= 500`.
If n is larger, precompute range sums (makes `mergeCost` O(1)) or apply Knuth's optimization.

---

## 4. Generic Template — Code

### Top-down (write this first — correct by construction)

Recursion handles the "shorter sub-ranges first" requirement automatically. Harder to mess up.

```cpp
const int MAXN = 505;
long long memo[MAXN][MAXN];
bool seen[MAXN][MAXN];

// Plug in your own mergeCost(l, k, r) and baseCase(l)
long long solve(int l, int r) {
    if (l == r) return baseCase(l);
    if (seen[l][r]) return memo[l][r];
    seen[l][r] = true;

    long long best = 1e18;            // use -1e18 for maximization
    for (int k = l; k < r; k++) {
        long long cand = solve(l, k) + solve(k + 1, r) + mergeCost(l, k, r);
        best = min(best, cand);       // or max
    }
    return memo[l][r] = best;
}
```

### Bottom-up (convert after logic is verified)

```cpp
int n;
long long dp[MAXN][MAXN];

// Initialize base case
for (int i = 0; i < n; i++) dp[i][i] = baseCase(i);

// Fill by increasing length
for (int len = 2; len <= n; len++) {
    for (int l = 0; l + len - 1 < n; l++) {
        int r = l + len - 1;
        dp[l][r] = 1e18;                      // or -1e18
        for (int k = l; k < r; k++) {
            long long cand = dp[l][k] + dp[k + 1][r] + mergeCost(l, k, r);
            dp[l][r] = min(dp[l][r], cand);
        }
    }
}
// Answer: dp[0][n-1]
```

**The only three things that change across problems:**
1. What `mergeCost(l, k, r)` is
2. What the split point `k` represents
3. Whether you need extra state beyond `[l][r]`

---

## 5. Problem 1 — Matrix Chain Multiplication

**Link:** [GFG — Matrix Chain Multiplication](https://www.geeksforgeeks.org/matrix-chain-multiplication-dp-8/)

### Problem
You have `n` matrices. Matrix `i` has dimensions `p[i-1] x p[i]`.
Find the minimum number of scalar multiplications to compute `A1 x A2 x ... x An`.

### Thinking Process
- Multiplying two matrices of size `(a x b)` and `(b x c)` costs `a * b * c` multiplications.
- We don't know the optimal parenthesization — so we try all of them.
- `dp[i][j]` = min cost to fully multiply matrices `i` through `j`.

### Key insight — "what multiplication happens last?"
If we split at `k`, the last step multiplies `(result of i..k)` and `(result of k+1..j)`.
- Result of `i..k` has dimensions `p[i-1] x p[k]`
- Result of `k+1..j` has dimensions `p[k] x p[j]`
- Cost of final multiplication = `p[i-1] * p[k] * p[j]`

### Recurrence
```
dp[i][j] = min over k in [i, j-1] of:
            dp[i][k] + dp[k+1][j] + p[i-1] * p[k] * p[j]

Base case: dp[i][i] = 0  (single matrix, no multiplication needed)
```

### Visual Trace
```
Matrices: A1(10x30), A2(30x5), A3(5x60)
p = [10, 30, 5, 60]   (n=3 matrices -> 4 entries)

dp[1][1] = dp[2][2] = dp[3][3] = 0

len=2:
  dp[1][2]: k=1 -> 0 + 0 + 10*30*5 = 1500
  dp[2][3]: k=2 -> 0 + 0 + 30*5*60 = 9000

len=3:
  dp[1][3]:
    k=1: dp[1][1] + dp[2][3] + p[0]*p[1]*p[3] = 0 + 9000 + 10*30*60 = 27000
    k=2: dp[1][2] + dp[3][3] + p[0]*p[2]*p[3] = 1500 + 0 + 10*5*60  =  4500  <- best
  dp[1][3] = 4500
```

### Code
```cpp
// p[] has n+1 entries: matrix i has dimensions p[i-1] x p[i]
long long matrixChain(vector<int>& p) {
    int n = p.size() - 1;  // number of matrices
    long long dp[105][105];
    memset(dp, 0, sizeof(dp));

    for (int len = 2; len <= n; len++) {
        for (int i = 1; i + len - 1 <= n; i++) {
            int j = i + len - 1;
            dp[i][j] = 1e18;
            for (int k = i; k < j; k++) {
                long long cost = dp[i][k] + dp[k + 1][j]
                                 + (long long)p[i - 1] * p[k] * p[j];
                dp[i][j] = min(dp[i][j], cost);
            }
        }
    }
    return dp[1][n];
}
```

> **Note:** Matrices are 1-indexed here. `dp[i][j]` uses `p[i-1]` as the left dimension.
> Always use `long long` — the product of three large dimensions overflows `int`.

---

## 6. Problem 2 — Boolean Parenthesization

**Link:** [GFG — Boolean Parenthesization](https://www.geeksforgeeks.org/boolean-parenthesization-problem-dp-37/)

### Problem
Given a boolean expression with `T`, `F` and operators `&`, `|`, `^`.
Count the number of ways to parenthesize it so it evaluates to `True`.

### Thinking Process
- Same split-point framework, but instead of one value per range, track **two counts**:
  - `T[l][r]` = ways to make `[l..r]` evaluate to true
  - `F[l][r]` = ways to make `[l..r]` evaluate to false
- For each split `k`, operator `op[k]` sits between symbols `k` and `k+1`.
- Combine left and right counts based on the operator.

### Key Observations
```
op = '&':  true  only if both sides true
           tWays = T[l][k] * T[k+1][r]
           fWays = (total) - tWays

op = '|':  false only if both sides false
           fWays = F[l][k] * F[k+1][r]
           tWays = (total) - fWays

op = '^':  true if exactly one side true
           tWays = T[l][k]*F[k+1][r] + F[l][k]*T[k+1][r]
           fWays = (total) - tWays
```

### Code
```cpp
long long countWays(string symbols, string operators) {
    int n = symbols.size();
    long long T[105][105] = {}, F[105][105] = {};

    for (int i = 0; i < n; i++) {
        T[i][i] = (symbols[i] == 'T') ? 1 : 0;
        F[i][i] = (symbols[i] == 'F') ? 1 : 0;
    }

    for (int len = 2; len <= n; len++) {
        for (int i = 0; i + len - 1 < n; i++) {
            int j = i + len - 1;
            T[i][j] = 0;
            F[i][j] = 0;
            for (int k = i; k < j; k++) {
                char op = operators[k];
                long long lt = T[i][k], lf = F[i][k];
                long long rt = T[k+1][j], rf = F[k+1][j];
                long long total = (lt + lf) * (rt + rf);

                long long tWays = 0, fWays = 0;
                if (op == '&') {
                    tWays = lt * rt;
                    fWays = total - tWays;
                } else if (op == '|') {
                    fWays = lf * rf;
                    tWays = total - fWays;
                } else {  // '^'
                    tWays = lt * rf + lf * rt;
                    fWays = total - tWays;
                }
                T[i][j] += tWays;
                F[i][j] += fWays;
            }
        }
    }
    return T[0][n - 1];
}
```

---

## 7. Problem 3 — Palindrome Partitioning II (LC 132)

**Link:** [LeetCode 132](https://leetcode.com/problems/palindrome-partitioning-ii/)

### Problem
Given string `s`, find the minimum number of cuts to partition it so every part is a palindrome.

### Thinking Process — Two DPs in One

**Part 1 (Interval DP):** Build `isPal[l][r]` — is `s[l..r]` a palindrome?
```
isPal[i][i] = true        (single character)
isPal[i][j] = (s[i] == s[j]) AND (isPal[i+1][j-1] OR len == 2)
```

**Part 2 (1D DP):** `cut[i]` = min cuts to partition `s[0..i]`.
```
If s[0..i] is palindrome -> cut[i] = 0
Otherwise: cut[i] = min over all j < i where s[j+1..i] is palindrome:
           cut[i] = cut[j] + 1
```

### Code
```cpp
int minCut(string s) {
    int n = s.size();
    bool isPal[1005][1005] = {};

    // Part 1: build palindrome table via interval DP
    for (int i = 0; i < n; i++) isPal[i][i] = true;
    for (int len = 2; len <= n; len++) {
        for (int i = 0; i + len - 1 < n; i++) {
            int j = i + len - 1;
            if (s[i] == s[j] && (len == 2 || isPal[i + 1][j - 1]))
                isPal[i][j] = true;
        }
    }

    // Part 2: 1D DP for minimum cuts
    vector<int> cut(n, INT_MAX);
    for (int i = 0; i < n; i++) {
        if (isPal[0][i]) {
            cut[i] = 0;
            continue;
        }
        for (int j = 0; j < i; j++) {
            if (isPal[j + 1][i] && cut[j] != INT_MAX)
                cut[i] = min(cut[i], cut[j] + 1);
        }
    }
    return cut[n - 1];
}
```

> **Pattern:** When a problem needs "is this range a palindrome?", always precompute the interval DP table first, then use it as a lookup in the second pass.

---

## 8. Problem 4 — Slimes (AtCoder DP N)

**Link:** [AtCoder DP Contest — Problem N](https://atcoder.jp/contests/dp/tasks/dp_n)

### Problem
`n` slimes in a row with sizes `a[0..n-1]`. Merge two adjacent slimes: cost = sum of their sizes.
Find the minimum total cost to merge all into one.

### Thinking Process
This is **MCM with a different `mergeCost`**.

When you merge group `[l..r]` into one slime, you pay `sum(a[l..r])` for that final merge.
So `dp[l][r]` = min total cost to merge `a[l..r]` into one slime.

Precompute prefix sums so `sum(l, r)` is O(1) inside the triple loop (otherwise O(n^4)).

### Why you always pay rangeSum:
No matter how you order the intermediate merges, the final merge always combines two groups that together span `[l..r]`. The combined weight they produce = `sum(l, r)`. So every possible split `k` pays the same `rangeSum` — only the internal costs of the two halves differ.

### Code
```cpp
long long minCostSlimes(vector<long long>& a) {
    int n = a.size();
    vector<long long> pre(n + 1, 0);
    for (int i = 0; i < n; i++) pre[i + 1] = pre[i] + a[i];

    long long dp[505][505] = {};

    for (int len = 2; len <= n; len++) {
        for (int l = 0; l + len - 1 < n; l++) {
            int r = l + len - 1;
            long long rangeSum = pre[r + 1] - pre[l];
            dp[l][r] = 1e18;
            for (int k = l; k < r; k++)
                dp[l][r] = min(dp[l][r], dp[l][k] + dp[k + 1][r] + rangeSum);
        }
    }
    return dp[0][n - 1];
}
```

---

## 9. Problem 5 — Removal Game / Predict the Winner

**Link (Removal Game):** [CSES — Removal Game](https://cses.fi/problemset/task/1097)
**Link (Predict the Winner):** [LeetCode 486](https://leetcode.com/problems/predict-the-winner/)

### Problem
Array of values. Two players take turns picking from **either end**.
Each player plays optimally to maximize their own score. Can player 1 win?

### The Score Difference Trick

Naive: `dp[l][r]` = max score player 1 can get from `[l..r]`. **This breaks** because you don't know whose turn it is on sub-ranges.

**Fix:** Define `dp[l][r]` = **max score difference (current player - other player) from `a[l..r]`**.

Why this works:
- Current player picks `a[l]` -> other player becomes current on `[l+1..r]`
- Other player's best difference = `dp[l+1][r]`
- So OUR net from picking left = `a[l] - dp[l+1][r]`
- Similarly, picking right gives `a[r] - dp[l][r-1]`

```
dp[l][r] = max(a[l] - dp[l+1][r],    // pick left
               a[r] - dp[l][r-1])     // pick right

Base: dp[i][i] = a[i]
Player 1 wins if dp[0][n-1] >= 0
```

### Code
```cpp
// LC 486: returns true if player 1 can win or tie
bool predictWinner(vector<int>& nums) {
    int n = nums.size();
    int dp[505][505] = {};

    for (int i = 0; i < n; i++) dp[i][i] = nums[i];

    for (int len = 2; len <= n; len++) {
        for (int l = 0; l + len - 1 < n; l++) {
            int r = l + len - 1;
            dp[l][r] = max(nums[l] - dp[l + 1][r],
                           nums[r] - dp[l][r - 1]);
        }
    }
    return dp[0][n - 1] >= 0;
}

// CSES Removal Game: get player 1's actual score
long long removalGame(vector<long long>& a) {
    int n = a.size();
    long long dp[505][505] = {};

    for (int i = 0; i < n; i++) dp[i][i] = a[i];

    for (int len = 2; len <= n; len++) {
        for (int l = 0; l + len - 1 < n; l++) {
            int r = l + len - 1;
            dp[l][r] = max(a[l] - dp[l + 1][r],
                           a[r] - dp[l][r - 1]);
        }
    }
    long long total = 0;
    for (long long x : a) total += x;
    return (total + dp[0][n - 1]) / 2;  // player 1's actual total
}
```

---

## 10. Problem 6 — Burst Balloons (LC 312)

**Link:** [LeetCode 312](https://leetcode.com/problems/burst-balloons/)

### Problem
Array of balloons with values. Bursting balloon `i` earns `nums[i-1] * nums[i] * nums[i+1]`.
After bursting, neighbors close in. Maximize total coins.

### The Core Mistake People Make

**Wrong thinking:** "Which balloon do I burst first in range `[l, r]`?"

When you burst a balloon first, its neighbors change — the sub-problems become dependent on each other. The recurrence doesn't close cleanly.

### The "Aha" Moment — Think in Reverse

**Right thinking:** "Which balloon in range `(l, r)` do I burst **last**?"

If balloon `k` is the **last** one burst in open range `(l, r)`:
- Everything to its left in `(l, k)` is already gone.
- Everything to its right in `(k, r)` is already gone.
- Its only remaining neighbors are the **boundary balloons** `nums[l]` and `nums[r]`.
- Cost of bursting `k` last = `nums[l] * nums[k] * nums[r]`.
- Sub-problems `(l, k)` and `(k, r)` are now fully independent!

### Boundary trick
Pad `nums` with `1` on both sides so boundaries always exist even at the array edges.

### Recurrence
```
dp[l][r] = max coins from bursting all balloons strictly inside (l, r)
           (l and r are EXCLUSIVE -- those balloons are still alive as boundaries)

dp[l][r] = max over k in (l, r):
            dp[l][k] + dp[k][r] + a[l] * a[k] * a[r]

Note: k starts from l+1, not l (since l is a boundary, not inside)
```

### Visual
```
nums = [3, 1, 5, 8], padded a = [1, 3, 1, 5, 8, 1]

dp[0][5] = answer (all balloons inside open range (0, 5))

Try k=1 (balloon with value 3) as last burst:
  Left part:  dp[0][1] = 0   (nothing inside (0,1))
  Right part: dp[1][5] = max coins from (1,5) without balloon 1
  Final cost: a[0]*a[1]*a[5] = 1*3*1 = 3

Try k=2 (value 1), k=3 (value 5), k=4 (value 8)... take the max.
```

### Code
```cpp
int maxCoins(vector<int>& nums) {
    int n = nums.size();
    vector<int> a(n + 2);
    a[0] = a[n + 1] = 1;
    for (int i = 0; i < n; i++) a[i + 1] = nums[i];

    int sz = n + 2;
    int dp[305][305] = {};

    for (int len = 2; len <= sz - 1; len++) {
        for (int l = 0; l + len <= sz - 1; l++) {
            int r = l + len;
            for (int k = l + 1; k < r; k++) {
                int cand = dp[l][k] + dp[k][r] + a[l] * a[k] * a[r];
                dp[l][r] = max(dp[l][r], cand);
            }
        }
    }
    return dp[0][n + 1];
}
```

> **Key difference from normal interval DP:** `dp[l][r]` means the open interval `(l, r)`.
> Balloons at `l` and `r` are alive as boundaries — that's why `k` starts from `l+1`.

---

## 11. Problem 7 — Min Cost Tree from Leaf Values (LC 1130)

**Link:** [LeetCode 1130](https://leetcode.com/problems/minimum-cost-tree-from-leaf-values/)

### Problem
Given leaf values `arr`, build a binary tree where leaves are `arr[0..n-1]` in order.
Each internal node's value = product of max leaves in its left and right subtrees.
Minimize the sum of all internal node values.

### Thinking Process
- `dp[l][r]` = min sum of internal nodes built from leaves `arr[l..r]`.
- Split at `k`: left subtree uses `arr[l..k]`, right uses `arr[k+1..r]`.
- Internal node for this split = `max(arr[l..k]) * max(arr[k+1..r])`.
- Precompute `mx[l][r]` = max in `arr[l..r]` for O(1) lookup.

### Code
```cpp
int mctFromLeafValues(vector<int>& arr) {
    int n = arr.size();
    int mx[55][55] = {};
    long long dp[55][55] = {};

    // Precompute range max
    for (int i = 0; i < n; i++) mx[i][i] = arr[i];
    for (int len = 2; len <= n; len++) {
        for (int l = 0; l + len - 1 < n; l++) {
            int r = l + len - 1;
            mx[l][r] = max(mx[l][r - 1], arr[r]);
        }
    }

    // Interval DP
    for (int len = 2; len <= n; len++) {
        for (int l = 0; l + len - 1 < n; l++) {
            int r = l + len - 1;
            dp[l][r] = 1e18;
            for (int k = l; k < r; k++) {
                long long cost = dp[l][k] + dp[k + 1][r]
                                 + (long long)mx[l][k] * mx[k + 1][r];
                dp[l][r] = min(dp[l][r], cost);
            }
        }
    }
    return (int)dp[0][n - 1];
}
```

---

## 12. Problem 8 — Strange Printer (LC 664)

**Link:** [LeetCode 664](https://leetcode.com/problems/strange-printer/)

### Problem
A printer prints a contiguous run of the **same character** each turn, overwriting previous output.
Find the minimum turns to print string `s`.

### Thinking Process
- `dp[i][j]` = min turns to print `s[i..j]`.
- Base: `dp[i][i] = 1`.
- Default: `dp[i][j] = dp[i][j-1] + 1` (print `s[j]` separately as one extra turn).
- **Optimization:** If `s[k] == s[j]` for some `k` in `[i, j-1]`, we can extend the print run for `s[k]` to also cover `s[j]` in the same turn, effectively getting `s[j]` for free — we just need to print the interior `s[k+1..j-1]` separately in between.

```
When s[k] == s[j]:
  dp[i][j] = min(dp[i][j], dp[i][k] + dp[k+1][j-1])
  (interior dp[k+1][j-1] = 0 if k+1 > j-1)
```

### Code
```cpp
int strangePrinter(string s) {
    int n = s.size();
    int dp[105][105] = {};

    for (int i = n - 1; i >= 0; i--) {
        dp[i][i] = 1;
        for (int j = i + 1; j < n; j++) {
            dp[i][j] = dp[i][j - 1] + 1;  // print s[j] separately
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

## 13. Problem 9 — Optimal BST

**Link:** [GFG — Optimal BST](https://www.geeksforgeeks.org/optimal-binary-search-tree-dp-24/)

### Problem
Given sorted keys and their access frequencies, build a BST minimizing expected search cost.
Cost = sum of `freq[i] * depth[i]` over all keys.

### Thinking Process
- `dp[i][j]` = min cost BST from keys `i..j`.
- Try every key `r` in `[i, j]` as the **root**.
- Left subtree: `[i, r-1]`, Right subtree: `[r+1, j]`.
- Every key in the subtree has its depth increased by 1 due to the root -> we pay `sum(freq[i..j])` extra per merge level. This telescopes to: add `rangeSum(i, j)` once for each sub-call.

### Code
```cpp
int optimalBST(vector<int>& freq) {
    int n = freq.size();
    int pre[55] = {};
    for (int i = 0; i < n; i++) pre[i + 1] = pre[i] + freq[i];

    int dp[55][55] = {};
    for (int i = 0; i < n; i++) dp[i][i] = freq[i];

    for (int len = 2; len <= n; len++) {
        for (int i = 0; i + len - 1 < n; i++) {
            int j = i + len - 1;
            int rangeSum = pre[j + 1] - pre[i];
            dp[i][j] = INT_MAX;
            for (int r = i; r <= j; r++) {
                int leftCost  = (r > i) ? dp[i][r - 1] : 0;
                int rightCost = (r < j) ? dp[r + 1][j] : 0;
                dp[i][j] = min(dp[i][j], leftCost + rightCost + rangeSum);
            }
        }
    }
    return dp[0][n - 1];
}
```

---

## 14. Problem 10 — Remove Boxes (LC 546) [Hardest]

**Link:** [LeetCode 546](https://leetcode.com/problems/remove-boxes/)

### Problem
Array of colored boxes. Remove `k` consecutive same-colored boxes -> earn `k^2` points.
Maximize total points.

### Why `dp[l][r]` is not enough

You might want to **delay bursting** boxes of a certain color to "attach" more boxes of the same color together and earn a bigger `k^2` bonus. The cost of `[l, r]` depends on how many same-colored boxes are waiting **outside the range** to join later.

### State: `dp[l][r][k]`
= max score from `boxes[l..r]` when there are `k` extra boxes of color `boxes[r]` attached just to the right.

### Transition
```
Option 1: burst boxes[r] together with the k extra boxes right now
  -> score = (k+1)^2 + dp[l][r-1][0]

Option 2: find m in [l, r-1] where boxes[m] == boxes[r]
  -> attach boxes[r] to boxes[m], burst them together later
  -> dp[l][m][k+1] + dp[m+1][r-1][0]
```

### Code
```cpp
int memo[105][105][105];

int solve(vector<int>& boxes, int l, int r, int k) {
    if (l > r) return 0;
    if (memo[l][r][k] != -1) return memo[l][r][k];

    // Optimization: collapse trailing same-colored boxes at the right
    int rr = r, kk = k;
    while (rr > l && boxes[rr] == boxes[rr - 1]) {
        rr--;
        kk++;
    }

    // Option 1: burst them all now
    int res = (kk + 1) * (kk + 1) + solve(boxes, l, rr - 1, 0);

    // Option 2: find a matching box further left and delay burst
    for (int m = l; m < rr; m++) {
        if (boxes[m] == boxes[rr]) {
            res = max(res, solve(boxes, l, m, kk + 1)
                        + solve(boxes, m + 1, rr - 1, 0));
        }
    }

    return memo[l][r][k] = res;
}

int removeBoxes(vector<int>& boxes) {
    memset(memo, -1, sizeof(memo));
    return solve(boxes, 0, (int)boxes.size() - 1, 0);
}
```

---

## 15. Common Pitfalls

### Pitfall 1 — Wrong loop order (most common)
If you loop `l` as outer and `r` as inner, `dp[k+1][r]` for `k+1 > l` may not be ready.
**Fix:** Always loop by **increasing length** as the outermost loop.

### Pitfall 2 — Off-by-one in length loop
```cpp
for (int l = 0; l + len - 1 < n; l++)   // correct
// NOT: l + len < n  OR  l + len <= n
```

### Pitfall 3 — Empty range not handled
Add an explicit base case:
```cpp
if (l > r) return 0;
```
Never rely on uninitialized array being zero.

### Pitfall 4 — Integer overflow
`mergeCost` in MCM is a product of three large numbers. Always use `long long`:
```cpp
(long long)p[i - 1] * p[k] * p[j]
```

### Pitfall 5 — O(n^4) instead of O(n^3)
If `mergeCost(l, k, r)` itself loops over the range to sum values -> O(n^4) -> TLE.
Precompute prefix sums so it's O(1):
```cpp
long long rangeSum = pre[r + 1] - pre[l];
```

### Pitfall 6 — "First vs. Last" confusion (Burst Balloons)
Natural instinct: "which balloon do I burst first?" -> breaks independence.
Correct: "which balloon is burst **last** in this range?" -> sub-problems become independent.
When a removal problem's recurrence isn't closing cleanly, **flip first -> last**.

---

## 16. Knuth's Optimization (Bonus)

For problems where the optimal split point `opt[l][r]` is **monotone**:
```
opt[l][r-1] <= opt[l][r] <= opt[l+1][r]
```
Shrink the inner loop to `k` in `[opt[l][r-1], opt[l+1][r]]`, bringing time from O(n^3) to O(n^2).

Applies to classic MCM, Optimal BST, and any cost function satisfying the **quadrangle inequality**.
Use only when n > 500 forces you to optimize.

```cpp
int opt[MAXN][MAXN];
for (int i = 0; i < n; i++) { dp[i][i] = baseCase(i); opt[i][i] = i; }

for (int len = 2; len <= n; len++) {
    for (int l = 0; l + len - 1 < n; l++) {
        int r = l + len - 1;
        int lo = (l < r) ? opt[l][r - 1] : l;
        int hi = (l < r) ? opt[l + 1][r] : r;
        dp[l][r] = 1e18;
        for (int k = lo; k <= hi; k++) {
            long long cand = dp[l][k] + dp[k + 1][r] + mergeCost(l, k, r);
            if (cand < dp[l][r]) {
                dp[l][r] = cand;
                opt[l][r] = k;
            }
        }
    }
}
```

---

## 17. Practice Ladder

Work through in this order. Each problem builds on the previous one.

| # | Problem | Platform | What it teaches |
|---|---------|----------|----------------|
| 1 | [Matrix Chain Multiplication](https://www.geeksforgeeks.org/matrix-chain-multiplication-dp-8/) | GFG | The base template |
| 2 | [Slimes](https://atcoder.jp/contests/dp/tasks/dp_n) | AtCoder DP N | Same template, different `mergeCost` |
| 3 | [Predict the Winner](https://leetcode.com/problems/predict-the-winner/) | LC 486 | Score difference trick for 2-player |
| 4 | [Removal Game](https://cses.fi/problemset/task/1097) | CSES | Same 2-player trick, confirm transfer |
| 5 | [Palindrome Partitioning II](https://leetcode.com/problems/palindrome-partitioning-ii/) | LC 132 | Interval DP as preprocessing |
| 6 | [Min Cost Tree from Leaf Values](https://leetcode.com/problems/minimum-cost-tree-from-leaf-values/) | LC 1130 | Back to MCM shape, precompute max |
| 7 | [Burst Balloons](https://leetcode.com/problems/burst-balloons/) | LC 312 | "Think last, not first" |
| 8 | [Strange Printer](https://leetcode.com/problems/strange-printer/) | LC 664 | Non-obvious split semantics |
| 9 | [Boolean Parenthesization](https://www.geeksforgeeks.org/boolean-parenthesization-problem-dp-37/) | GFG | Two values (T/F) per state |
| 10 | [Remove Boxes](https://leetcode.com/problems/remove-boxes/) | LC 546 | Extra dimension when range is not enough |

### If you only have time for 4:
**MCM -> Slimes -> Predict the Winner -> Burst Balloons**

These 4 cover: base template, different mergeCost, game theory, and the "think in reverse" reframe.
The rest are variations on these ideas.

---

> **Final mental model:**
> Every interval DP problem is asking:
> *"What is the best way to fully resolve range `[l, r]`, given that I make one last decision?"*
>
> Identify what that last decision is — split point, last burst, last move from an end —
> and the recurrence writes itself.
