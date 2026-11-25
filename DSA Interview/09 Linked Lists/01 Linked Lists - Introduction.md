---
topic: linkedlist-intro
tags: [dsa, linkedlist]
status: complete
---
# Linked Lists - Introduction

## A. Introduction
Linear data structure where elements (nodes) connected via pointers/references, not stored contiguously.

## B. Node Structure
`java
static class Node {
  int data;
  Node next;
}
`

## C. Types
- Singly Linked List (SLL)
- Doubly Linked List (DLL) - prev + next
- Circular Linked List (CLL)
- Doubly Circular

## D. Memory Layout
Non-contiguous, dynamic size.

## E. Operations
| Op | SLL | Array |
|---|---|---|
| Access by index | O(n) | O(1) |
| Insert at head | O(1) | O(n) |
| Insert at tail | O(n) (or O(1) with tail) | O(1) |
| Insert middle | O(n) | O(n) |
| Delete head | O(1) | O(n) |
| Delete middle | O(n) | O(n) |

## F. Key Techniques
- Traversal
- Insertion/deletion
- Reversal (iterative/recursive)
- Middle node (slow/fast)
- Cycle detection (Floyd)
- Merge two lists
- Intersection
- Remove nth from end

## G. Java Example
Basic traversal.

## H. Interview Q&A
Q: Array vs LinkedList?
A: Array better for access; LL better for frequent head insert/delete (no shifting).

Q: Why fast/slow pointers?
A: Find middle/cycle in one pass efficiently.

## I. Revision
- Node-based
- Dynamic
- Extra memory for links
- No random access
