---
topic: fundamentals-intro
tags: [dsa, fundamentals]
status: complete
---
# Data Structures and Algorithms - Introduction

## A. Introduction

### What is a Data Structure?
A data structure is a way of organizing, storing, and managing data so it can be used efficiently. It defines the relationship between data elements and operations on them.

### What is an Algorithm?
An algorithm is a finite, well-defined sequence of steps to solve a problem or perform a task.

### What problem does it solve?
Poor data organization leads to slow programs. The right DS + algorithm gives efficient, scalable solutions.

### Why important?
Core to writing efficient code, cracking interviews, and building real systems.

### Real-world usage
Databases, compilers, operating systems, routing, caches, search engines, etc.

### Prerequisites
Basic Java syntax, variables, loops, arrays, functions.

## B. Fundamental Concepts

### Abstract Data Type (ADT)
ADT is a logical model - defines what operations are possible, not how implemented. Example: Stack ADT says push/pop/peek; implementation can be array or linked list.

### Linear vs Non-Linear
- **Linear**: Elements arranged sequentially (Arrays, Linked Lists, Stacks, Queues)
- **Non-Linear**: Hierarchical/connected (Trees, Graphs)

### Static vs Dynamic
- **Static**: Fixed size (Arrays in C, but Java arrays fixed; Strings behave specially)
- **Dynamic**: Can grow/shrink (ArrayList, LinkedList)

### Primitive vs Non-Primitive
- **Primitive**: int, char, boolean, float, double, byte, short, long
- **Non-Primitive**: Arrays, Classes, Interfaces, Collections

### Contiguous vs Linked Memory
- **Contiguous**: Elements stored together (Arrays) - fast access, costly insertion/deletion middle
- **Linked**: Elements stored separately with links (LinkedList) - flexible, extra memory for links

## C. Complexity - From First Principles

### Why analyze?
To predict performance as input grows (n -> large). We care about growth, not exact time.

### Time Complexity
Number of operations relative to input size n.

### Space Complexity
Total memory used = Input memory + Auxiliary memory. We usually care about auxiliary (extra) space.

### Asymptotic Notations
- **Big O (O)**: Upper bound - worst case growth (most common)
- **Big Omega (Ω)**: Lower bound - best case
- **Big Theta (Θ)**: Tight bound - when best==worst asymptotically

### Best, Average, Worst Case
Example Linear Search:
- Best: element at index 0 -> O(1)
- Average: middle -> O(n)
- Worst: not found/end -> O(n)

### Amortized Analysis
Average cost per operation over a sequence. Example: ArrayList add() - mostly O(1), occasionally O(n) on resize, amortized O(1).

### Recurrence Relations
T(n) = 2T(n/2) + O(n) etc - describe recursive algorithms.

### Recursion Tree Method
Draw tree of recursive calls, sum work per level.

### Master Theorem (common forms)
For T(n)=aT(n/b)+f(n), compare f(n) with n^(log_b a). Standard cases apply.

### Iterative vs Recursive
- Iterative: explicit loop, usually less stack memory
- Recursive: cleaner for tree/DFS/backtrack, uses call stack (risk stack overflow)

### In-place vs Not In-place
- In-place: O(1) extra space (e.g., swap in place)
- Not in-place: uses extra arrays/structures

### Stable vs Unstable
- Stable: equal elements keep original relative order (Merge sort)
- Unstable: order of equal elements may change (Quick sort typical)

### Trade-offs
Usually time-space trade-off. Faster often needs more memory (hash tables).

## D. Common Complexities with Examples
| Notation | Meaning | Example |
|---|---|---|
| O(1) | Constant | arr[0], push/pop stack |
| O(log n) | Logarithmic | Binary search (halving) |
| O(n) | Linear | Single loop |
| O(n log n) | Linearithmic | Merge sort |
| O(n^2) | Quadratic | Two nested loops |
| O(2^n) | Exponential | Power set naive recursion |
| O(n!) | Factorial | All permutations |

## E. Java Implementation Notes
Java has references (not raw pointers). Arrays are objects. Collections provide dynamic structures.

## F. Interview Theory Q&A (Essentials)
Q: Difference between data structure and algorithm?
A: DS = how we store data; Algorithm = how we process it.

Q: Why O(1) for array access?
A: Direct index calculation base + index*size.

Q: When to prefer recursion?
A: Tree traversals, divide and conquer, backtracking where state is natural.

## G. Coding Application
Identify pattern: access pattern, insert/delete needs, search needs guide choice.

## H. Revision
- ADT vs DS
- Linear vs Non-linear
- Best/Average/Worst
- Big O vs Theta vs Omega
- Stable/In-place
- Complexity classes
