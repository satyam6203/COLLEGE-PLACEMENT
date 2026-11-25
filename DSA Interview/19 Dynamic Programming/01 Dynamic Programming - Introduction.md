---
topic: dp-intro
tags: [dsa, dp]
status: complete
---
# Dynamic Programming - Introduction

## A. Introduction
DP optimizes by storing results of overlapping subproblems (memoization/tabulation). Uses optimal substructure.

## B. When to use DP
- Count ways
- Min/Max cost/path
- Longest/Shortest in terms of value with choices
- Overlapping subproblems
- Optimal substructure

## C. Approaches
- **Top-down (Memoization)**: Recursion + cache (HashMap/array)
- **Bottom-up (Tabulation)**: Iterative, build table from base
- **Space Optimization**: Use variables/rolling arrays

## D. Steps to solve DP
1. Identify if DP applicable
2. Define state (what dp[i] represents)
3. Find recurrence
4. Base cases
5. Compute order
6. Answer
7. Optimize space if possible

## E. Common Patterns
1D (Fibonacci, Climb Stairs), 2D/Grid, Knapsack, LCS/LIS, Coin Change, String DP, Interval DP, Tree DP, Bitmask DP.

## F. Memoization vs Tabulation
| Aspect | Memoization | Tabulation |
|---|---|---|
| Direction | Top-down | Bottom-up |
| Recursion | Yes | No |
| Stack | Uses call stack | Iterative |
| Easier to code | Often easier to derive | May need order |
| Computes all? | Only needed states | Usually all states |

## G. Example (Fibonacci)
Memo: dp[i]=dp[i-1]+dp[i-2] with cache
Tab: dp[0]=1,dp[1]=1, build up

## H. Interview Q&A
Q: What makes problem DP?
A: Overlapping subproblems + optimal substructure.

Q: How to identify state?
A: Parameters that uniquely define subproblem.

## I. Revision
- Overlap + optimal substructure
- State + transition
- Memoization vs tabulation
- Space optimize
