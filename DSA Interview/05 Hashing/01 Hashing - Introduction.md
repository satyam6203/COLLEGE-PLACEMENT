---
topic: hashing-intro
tags: [dsa, hashing]
status: complete
---
# Hashing - Introduction

## A. Introduction
Hashing maps key -> value using hash function for fast lookup.

## B. Concepts
### Hash Table
Array of buckets. Key passed to hashCode() -> index.

### Hash Function
Should be deterministic, uniform distribution, efficient.

### Collisions
Two different keys give same hash/index.

### Collision Resolution
- **Separate Chaining**: Each bucket is list/tree of entries
- **Open Addressing**: Find next slot (linear probing, quadratic probing, double hashing)

### Load Factor
(load/ capacity). When exceeded, resize (rehash).

### Java HashMap Internals (Key Points)
- Uses array of Node
- put/get use hash & index
- If collision, linked list; treeified if threshold in Java 8+
- Initial capacity/default load factor 0.75

### equals() and hashCode() Contract
Critical for custom keys:
- equal objects -> same hashCode
- same hashCode -> not necessarily equal
- consistent during use

## C. HashSet vs HashMap
HashSet backed by HashMap.

## D. Complexity
Avg O(1) for get/put, worst O(n) if all collide badly.

## E. Java Example
`java
HashMap<String,Integer> map = new HashMap<>();
map.put("a",1);
map.getOrDefault("b",0);
`

## F. Interview Q&A
Q: How HashMap works?
A: hash -> index -> bucket, check equals on collision.

Q: Why override both?
A: To find correct bucket and correct key.

Q: What if bad hashCode?
A: More collisions -> worse performance.

## G. Revision
- Hash function + buckets
- Chaining vs open addressing
- Load factor & rehash
- equals/hashCode contract
