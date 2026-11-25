---
topic: stack-queue-intro
tags: [dsa, stack, queue]
status: complete
---
# Stacks and Queues - Introduction

## A. Stacks (LIFO)
Last In First Out. push, pop, peek, isEmpty.

### Uses
Undo/Redo, Expression evaluation, Call stack, Backtracking

### Implementations
Array-based or LinkedList-based. In Java prefer ArrayDeque as stack.

### Java
`java
ArrayDeque<Integer> stack = new ArrayDeque<>();
stack.push(10); stack.pop(); stack.peek();
`

## B. Queues (FIFO)
First In First Out. offer/enqueue, poll/dequeue, peek.

### Types
Simple, Circular, Deque, Priority Queue

### Java
`java
ArrayDeque<Integer> q = new ArrayDeque<>();
q.offer(10); q.poll();
`

## C. Deque
Double Ended Queue - insert/delete from both ends.

## D. Monotonic Stack/Queue (intro)
Useful for "next greater/smaller", "sliding window max/min".

## E. Expression Notation
Infix, Prefix (Polish), Postfix (Reverse Polish). Conversion/evaluation concepts.

## F. Interview Q&A
Q: Stack vs Queue?
A: LIFO vs FIFO.

Q: Why ArrayDeque over Stack?
A: ArrayDeque faster, no legacy overhead, implements Deque.

## G. Revision
- LIFO/FIFO
- push/pop/peek vs offer/poll/peek
- ArrayDeque preferred
