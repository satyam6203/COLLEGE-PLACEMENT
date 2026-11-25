---
topic: strings-intro
tags: [dsa, strings]
status: complete
---
# Strings - Introduction

## A. Introduction
String is a sequence of characters.

## B. Concepts
### Immutability in Java
String objects immutable - any modification creates new String.

### String Pool
String literals stored in pool for reuse (String interning). 
ew String() creates heap object.

### String vs StringBuilder vs StringBuffer
| Feature | String | StringBuilder | StringBuffer |
|---|---|---|---|
| Mutable | No | Yes | Yes |
| Thread-safe | Immutable safe | No | Yes (synchronized) |
| Performance | Slow for concat | Fast | Slower than StringBuilder |
| Use case | Constants | Single thread | Multi-thread |

### Character encoding basics
ASCII, Unicode basics (interview level awareness).

## C. Key Operations
length(), charAt(), substring(), indexOf(), equals(), compareTo().

## D. Substring vs Subsequence
- **Substring**: contiguous sequence
- **Subsequence**: can skip, order preserved

## E. Pattern Matching ideas
Naive O(mn), KMP O(m+n), Rabin-Karp uses hashing.

## F. Anagrams, Palindromes
Common patterns - frequency maps, two pointers.

## G. Java Examples
`java
String s = "abc";
StringBuilder sb = new StringBuilder();
sb.append("a").append("b");
String res = sb.toString();
`

## H. Interview Q&A
Q: Why immutable?
A: Security, String pool, thread-safety, caching hashCode.

Q: String s="a"+"b" vs new StringBuilder?
A: Compiler may optimize literals, but loops need StringBuilder.

Q: equals() vs ==?
A: == reference, equals() content (for String).

## I. Revision
- Immutable
- Pool vs heap
- StringBuilder for concatenation in loops
- equals vs ==
