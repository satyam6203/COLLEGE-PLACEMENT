---
topic: complexity-analysis
tags: [dsa, complexity, big-o]
status: complete
---
# Time and Space Complexity - Detailed Theory

## A. Introduction
Why measure complexity? To compare algorithms independent of machine, language, or implementation details.

## B. Time Complexity from First Principles
Count primitive operations. Drop constants, drop lower order terms.

Example: sum of array
`java
int sum(int[] arr){
  int s=0; // 1
  for(int i=0;i<arr.length;i++) s += arr[i]; // n iterations
  return s; // 1
}
`
T(n)=n+2 -> O(n)

## C. Space Complexity
Space = auxiliary space + input space. Focus on auxiliary.

## D. Notations Explained
- O(g(n)): exists c,n0 such that f(n)<=c*g(n) for all n>=n0
- Ω(g(n)): lower bound
- Θ(g(n)): tight bound

## E. Growth Comparison
n vs log n matters as n grows large.

## F. Recurrence Relations
T(1)=1, T(n)=T(n/1/2)+1 -> O(log n) for binary search.

## G. Master Theorem
T(n)=aT(n/b)+f(n), a>=1,b>1
- if f(n)=O(n^(log_b a - ε)) -> T(n)=Θ(n^log_b a)
- if f(n)=Θ(n^log_b a) -> T(n)=Θ(n^log_b a log n)
- if f(n)=Ω(n^(log_b a + ε)) and regularity -> T(n)=Θ(f(n))

## H. Amortized Analysis
Dynamic array append: total O(n) for n appends -> amortized O(1).

## I. Java Notes
Recursion depth limit (StackOverflowError for deep recursion). Consider iterative or note constraints.

## J. Interview Q&A
Q: Why not measure actual time?
A: Hardware/OS vary; asymptotic shows growth.

Q: Difference between auxiliary and total space?
A: Auxiliary = extra space used by algo; total = includes input.
