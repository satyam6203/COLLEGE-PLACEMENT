---
topic: arrays-intro
tags: [dsa, arrays]
status: complete
---
# Arrays - Introduction

## A. Introduction
### What is an Array?
An array is a collection of elements of same data type stored in contiguous memory locations, accessed by index.

### Why introduced?
Efficient random access by index.

### Problem solved?
Need fast lookup by position without traversing structure.

### Real world
Images (2D arrays), tables, caches.

## B. Concepts
- **Indexing**: 0-based in Java
- **Contiguous memory**: allows random access
- **Homogeneous**: same type (primitive arrays)
- **Fixed size**: array length fixed at creation
- **Random access**: O(1)

## C. Memory Representation
Base address + index*element size.

## D. Operations & Costs
| Op | Time | Notes |
|---|---|---|
| Access arr[i] | O(1) | Direct |
| Traverse | O(n) | |
| Search (unsorted) | O(n) | Linear |
| Search (sorted) | O(log n) | Binary |
| Insert at end | O(1) amortized (if space) | |
| Insert at middle | O(n) | Shift elements |
| Delete from middle | O(n) | Shift elements |

## E. Multidimensional Arrays
2D arrays - array of arrays in Java.

## F. Dynamic Arrays
ArrayList uses array internally, resizes (grow) when full - amortized O(1) append.

## G. Subarrays vs Subsequences
- **Subarray (contiguous)**: arr[i...j]
- **Subsequence (not necessarily contiguous)**: order preserved

## H. Java Implementation
`java
int[] arr = new int[5];
arr[0] = 10;
// traversal
for(int x : arr) System.out.print(x+" ");
`

## I. Interview Q&A
Q: Why O(1) access?
A: arr[i] = base + i*size

Q: Why insert middle O(n)?
A: Need to shift elements right.

Q: Array vs ArrayList?
A: Array fixed, primitive allowed; ArrayList dynamic, objects/wrapper.

## J. Revision
- Contiguous, index-based
- Random access O(1)
- Middle ops O(n)
- Subarray vs subsequence
