# Dynamic Programming (DP)

Top 75 LeetCode Dynamic Programming (DP) problems for Java DSA interview preparation. Covers 1D DP, grid DP, subsequences, knapsack, LIS, strings, interval DP, tree DP, and bitmask DP. Focus on recognizing the DP state, transition, base case, and optimization.

## Table of Contents

- [Easy — 15 Questions](#easy--15-questions)
- [Medium — 40 Questions](#medium--40-questions)
  - [A. 1D DP and Decision-Based DP](#a-1d-dp-and-decision-based-dp)
  - [B. Grid DP and Path Problems](#b-grid-dp-and-path-problems)
  - [C. Subsequences, LIS and Partitioning](#c-subsequences-lis-and-partitioning)
  - [D. String DP](#d-string-dp)
  - [E. Stock, Interval and Tree DP](#e-stock-interval-and-tree-dp)
- [Hard — 20 Questions](#hard--20-questions)
- [Priority Recommendations](#priority-recommendations)
- [DP Patterns to Master](#dp-patterns-to-master)
- [Recommended Solving Order](#recommended-solving-order)

## Easy — 15 Questions

| # | Problem | Main Concept | LeetCode Link |
|---|---------|--------------|---------------|
| 1 | Fibonacci Number | Basic 1D DP | [Link](https://leetcode.com/problems/fibonacci-number/) |
| 2 | Climbing Stairs | 1D DP | [Link](https://leetcode.com/problems/climbing-stairs/) |
| 3 | Min Cost Climbing Stairs | Minimum-cost DP | [Link](https://leetcode.com/problems/min-cost-climbing-stairs/) |
| 4 | N-th Tribonacci Number | Multiple previous states | [Link](https://leetcode.com/problems/n-th-tribonacci-number/) |
| 5 | Maximum Subarray | Kadane's algorithm / DP | [Link](https://leetcode.com/problems/maximum-subarray/) |
| 6 | Pascal's Triangle | State from previous row | [Link](https://leetcode.com/problems/pascals-triangle/) |
| 7 | Pascal's Triangle II | Space-optimized DP | [Link](https://leetcode.com/problems/pascals-triangle-ii/) |
| 8 | Range Sum Query – Immutable | Prefix sums | [Link](https://leetcode.com/problems/range-sum-query-immutable/) |
| 9 | Counting Bits | Bitwise DP | [Link](https://leetcode.com/problems/counting-bits/) |
| 10 | Is Subsequence | Two pointers / DP | [Link](https://leetcode.com/problems/is-subsequence/) |
| 11 | Unique Paths | Grid DP | [Link](https://leetcode.com/problems/unique-paths/) |
| 12 | Unique Paths II | Grid DP with obstacles | [Link](https://leetcode.com/problems/unique-paths-ii/) |
| 13 | Minimum Path Sum | Grid minimum-cost DP | [Link](https://leetcode.com/problems/minimum-path-sum/) |
| 14 | Triangle | Bottom-up DP | [Link](https://leetcode.com/problems/triangle/) |
| 15 | Best Time to Buy and Sell Stock | State tracking / DP | [Link](https://leetcode.com/problems/best-time-to-buy-and-sell-stock/) |

## Medium — 40 Questions

### A. 1D DP and Decision-Based DP

| # | Problem | Main Concept | LeetCode Link |
|---|---------|--------------|---------------|
| 16 | House Robber | Take or skip | [Link](https://leetcode.com/problems/house-robber/) |
| 17 | House Robber II | Circular DP | [Link](https://leetcode.com/problems/house-robber-ii/) |
| 18 | Delete and Earn | Transform to House Robber | [Link](https://leetcode.com/problems/delete-and-earn/) |
| 19 | Decode Ways | Counting DP | [Link](https://leetcode.com/problems/decode-ways/) |
| 20 | Word Break | Prefix DP | [Link](https://leetcode.com/problems/word-break/) |
| 21 | Integer Break | Partition DP | [Link](https://leetcode.com/problems/integer-break/) |
| 22 | Perfect Squares | Minimum-count DP | [Link](https://leetcode.com/problems/perfect-squares/) |
| 23 | Coin Change | Unbounded knapsack | [Link](https://leetcode.com/problems/coin-change/) |
| 24 | Coin Change II | Counting combinations | [Link](https://leetcode.com/problems/coin-change-ii/) |
| 25 | Combination Sum IV | Counting ordered sequences | [Link](https://leetcode.com/problems/combination-sum-iv/) |

### B. Grid DP and Path Problems

| # | Problem | Main Concept | LeetCode Link |
|---|---------|--------------|---------------|
| 26 | Unique Paths III | Backtracking + state | [Link](https://leetcode.com/problems/unique-paths-iii/) |
| 27 | Out of Boundary Paths | Grid counting DP | [Link](https://leetcode.com/problems/out-of-boundary-paths/) |
| 28 | Minimum Falling Path Sum | Grid minimum-cost DP | [Link](https://leetcode.com/problems/minimum-falling-path-sum/) |
| 29 | Maximal Square | 2D DP | [Link](https://leetcode.com/problems/maximal-square/) |
| 30 | Count Square Submatrices with All Ones | 2D DP | [Link](https://leetcode.com/problems/count-square-submatrices-with-all-ones/) |
| 31 | Cherry Pickup II | Multi-agent grid DP | [Link](https://leetcode.com/problems/cherry-pickup-ii/) |
| 32 | Dungeon Game | Reverse grid DP | [Link](https://leetcode.com/problems/dungeon-game/) |

### C. Subsequences, LIS and Partitioning

| # | Problem | Main Concept | LeetCode Link |
|---|---------|--------------|---------------|
| 33 | Longest Increasing Subsequence | LIS | [Link](https://leetcode.com/problems/longest-increasing-subsequence/) |
| 34 | Number of Longest Increasing Subsequence | LIS + counting | [Link](https://leetcode.com/problems/number-of-longest-increasing-subsequence/) |
| 35 | Longest Arithmetic Subsequence | DP with differences | [Link](https://leetcode.com/problems/longest-arithmetic-subsequence/) |
| 36 | Partition Equal Subset Sum | 0/1 knapsack | [Link](https://leetcode.com/problems/partition-equal-subset-sum/) |
| 37 | Target Sum | Subset-sum transformation | [Link](https://leetcode.com/problems/target-sum/) |
| 38 | Last Stone Weight II | Knapsack / partition | [Link](https://leetcode.com/problems/last-stone-weight-ii/) |
| 39 | Ones and Zeroes | 2D 0/1 knapsack | [Link](https://leetcode.com/problems/ones-and-zeroes/) |
| 40 | Longest Turbulent Subarray | State-based 1D DP | [Link](https://leetcode.com/problems/longest-turbulent-subarray/) |

### D. String DP

| # | Problem | Main Concept | LeetCode Link |
|---|---------|--------------|---------------|
| 41 | Longest Common Subsequence | 2D string DP | [Link](https://leetcode.com/problems/longest-common-subsequence/) |
| 42 | Edit Distance | Insert / delete / replace | [Link](https://leetcode.com/problems/edit-distance/) |
| 43 | Longest Palindromic Subsequence | Interval / subsequence DP | [Link](https://leetcode.com/problems/longest-palindromic-subsequence/) |
| 44 | Palindromic Substrings | Interval DP / expansion | [Link](https://leetcode.com/problems/palindromic-substrings/) |
| 45 | Longest Palindromic Substring | String DP / expansion | [Link](https://leetcode.com/problems/longest-palindromic-substring/) |
| 46 | Distinct Subsequences | Counting subsequences | [Link](https://leetcode.com/problems/distinct-subsequences/) |
| 47 | Interleaving String | 2D string DP | [Link](https://leetcode.com/problems/interleaving-string/) |
| 48 | Regular Expression Matching | String matching DP | [Link](https://leetcode.com/problems/regular-expression-matching/) |

### E. Stock, Interval and Tree DP

| # | Problem | Main Concept | LeetCode Link |
|---|---------|--------------|---------------|
| 49 | Best Time to Buy and Sell Stock with Cooldown | State-machine DP | [Link](https://leetcode.com/problems/best-time-to-buy-and-sell-stock-with-cooldown/) |
| 50 | Best Time to Buy and Sell Stock with Transaction Fee | State-machine DP | [Link](https://leetcode.com/problems/best-time-to-buy-and-sell-stock-with-transaction-fee/) |
| 51 | Maximum Product Subarray | Track max and min | [Link](https://leetcode.com/problems/maximum-product-subarray/) |
| 52 | Longest Valid Parentheses | 1D DP | [Link](https://leetcode.com/problems/longest-valid-parentheses/) |
| 53 | Unique Binary Search Trees | Catalan / interval DP | [Link](https://leetcode.com/problems/unique-binary-search-trees/) |
| 54 | House Robber III | Tree DP | [Link](https://leetcode.com/problems/house-robber-iii/) |
| 55 | Maximum Length of Pair Chain | DP / greedy | [Link](https://leetcode.com/problems/maximum-length-of-pair-chain/) |

## Hard — 20 Questions

| # | Problem | Main Concept | LeetCode Link |
|---|---------|--------------|---------------|
| 56 | Burst Balloons | Interval DP | [Link](https://leetcode.com/problems/burst-balloons/) |
| 57 | Strange Printer | Interval DP | [Link](https://leetcode.com/problems/strange-printer/) |
| 58 | Regular Expression Matching | String DP (Revisit) | [Link](https://leetcode.com/problems/regular-expression-matching/) |
| 59 | Wildcard Matching | String DP | [Link](https://leetcode.com/problems/wildcard-matching/) |
| 60 | Distinct Subsequences | Counting DP (Revisit) | [Link](https://leetcode.com/problems/distinct-subsequences/) |
| 61 | Scramble String | 3D / Partition DP | [Link](https://leetcode.com/problems/scramble-string/) |
| 62 | Longest Valid Parentheses | String DP (Revisit) | [Link](https://leetcode.com/problems/longest-valid-parentheses/) |
| 63 | Maximum Profit in Job Scheduling | DP + Binary search | [Link](https://leetcode.com/problems/maximum-profit-in-job-scheduling/) |
| 64 | Freedom Trail | State-transition DP | [Link](https://leetcode.com/problems/freedom-trail/) |
| 65 | Number of Music Playlists | Counting DP | [Link](https://leetcode.com/problems/number-of-music-playlists/) |
| 66 | Profitable Schemes | 3D knapsack DP | [Link](https://leetcode.com/problems/profitable-schemes/) |
| 67 | Shortest Common Supersequence | LCS-based DP | [Link](https://leetcode.com/problems/shortest-common-supersequence/) |
| 68 | Maximum Sum of 3 Non-Overlapping Subarrays | DP + Prefix sums | [Link](https://leetcode.com/problems/maximum-sum-of-3-non-overlapping-subarrays/) |
| 69 | Concatenated Words | Word-break DP | [Link](https://leetcode.com/problems/concatenated-words/) |
| 70 | Number of Ways to Form a Target String Given a Dictionary | Counting DP | [Link](https://leetcode.com/problems/number-of-ways-to-form-a-target-string-given-a-dictionary/) |
| 71 | Maximum Number of Ways to Partition an Array | Prefix sums + DP-style counting | [Link](https://leetcode.com/problems/maximum-number-of-ways-to-partition-an-array/) |
| 72 | Maximum Students Taking Exam | Bitmask DP | [Link](https://leetcode.com/problems/maximum-students-taking-exam/) |
| 73 | Parallel Courses II | Bitmask DP | [Link](https://leetcode.com/problems/parallel-courses-ii/) |
| 74 | Valid Permutations for DI Sequence | Counting / Permutation DP | [Link](https://leetcode.com/problems/valid-permutations-for-di-sequence/) |
| 75 | Number of Ways to Rearrange Sticks With K Sticks Visible | Combinatorial DP | [Link](https://leetcode.com/problems/number-of-ways-to-rearrange-sticks-with-k-sticks-visible/) |

## Priority Recommendations

**Easy Priority:** Climbing Stairs, Min Cost Climbing Stairs, Maximum Subarray, Unique Paths, Minimum Path Sum, and Best Time to Buy and Sell Stock.

**Medium Priority:** 16, 20, 23, 24, 29, 33, 36, 37, 41, 42, 46, 49, 51, and 54. These problems teach you how to move from simple recurrence relations to multidimensional and state-based DP.

## DP Patterns to Master

- [ ] **1D DP** - Climbing Stairs, House Robber
- [ ] **Grid / 2D DP** - Unique Paths, Minimum Path Sum
- [ ] **0/1 Knapsack** - Partition Equal Subset Sum, Target Sum
- [ ] **Unbounded Knapsack** - Coin Change, Coin Change II
- [ ] **LIS** - Longest Increasing Subsequence
- [ ] **String DP** - LCS, Edit Distance, Distinct Subsequences
- [ ] **State-machine DP** - Stock Buy and Sell problems
- [ ] **Interval DP** - Burst Balloons, Strange Printer
- [ ] **Tree DP** - House Robber III
- [ ] **Bitmask DP** - Maximum Students Taking Exam

## Recommended Solving Order

1. **Days 1–3:** Basic 1D DP and grid DP
2. **Days 4–6:** House Robber, Coin Change, and knapsack
3. **Days 7–9:** LIS and string DP
4. **Days 10–11:** Stock DP, tree DP, and interval DP
5. **Days 12 onward:** Hard DP, bitmask DP, and mixed interview problems

**For each problem, try to identify these four things before coding:**
- **State:** What does dp[i] or dp[i][j] represent?
- **Transition:** How do previous states produce the current state?
- **Base case:** What are the smallest subproblems?
- **Optimization:** Can you reduce time or space complexity?

**Suggested starting path:** Start with Climbing Stairs ? House Robber ? Coin Change ? Partition Equal Subset Sum ? Longest Increasing Subsequence ? Longest Common Subsequence ? Edit Distance. This sequence builds the foundation for many of the harder problems on the list.
