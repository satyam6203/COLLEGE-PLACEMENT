---
topic: complexity-analysis-questions
tags: [dsa, theory, complexity]
---
# Complexity Analysis Questions

## Basic
### Q1. What is Big O Notation? Why do we use it?
**Answer:** Big O describes asymptotic upper bound - how runtime/space grows with input. We use it to compare algorithms independent of hardware.

### Q2. Difference between Big O, Big Omega, Big Theta?
**Answer:** O = upper bound (worst case focus), Ω = lower bound, Θ = tight bound (same asymptotically).

### Q3. What is the difference between best, average, worst case?
**Answer:** Best = minimum operations, Average = expected over inputs, Worst = maximum operations.

### Q4. Explain time vs space complexity with example.
**Answer:** Time = operations count; Space = memory used. Example: sum array O(n) time, O(1) space.

### Q5. What is amortized complexity?
**Answer:** Average per operation over sequence of operations (e.g. ArrayList resize).

## Intermediate
### Q6. Why is binary search O(log n)?
**Answer:** Each comparison halves search space: n/2^k -> k = log2(n).

### Q7. Why is merge sort O(n log n)?
**Answer:** log n levels, O(n) work per level.

### Q8. Why can quicksort be O(n^2)?
**Answer:** Poor pivot (already sorted) gives unbalanced partitions.

### Q9. Difference between auxiliary space and total space?
**Answer:** Auxiliary = extra space by algorithm; Total = input + auxiliary.

### Q10. When is recursion worse than iteration?
**Answer:** Deep recursion can cause stack overflow; iteration uses less stack.
