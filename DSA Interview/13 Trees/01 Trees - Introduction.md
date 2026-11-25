---
topic: trees-intro
tags: [dsa, trees]
status: complete
---
# Trees - Introduction

## A. Introduction
Hierarchical data structure with root and subtrees, connected acyclic graph in structure.

## B. Terminology
Root, Node, Edge, Parent/Child, Sibling, Leaf, Subtree, Height, Depth, Level, Degree, Path, Ancestor/Descendant.

## C. Types
General tree, Binary tree, Binary Search Tree, Complete/Full/Perfect, Skewed, Balanced.

## D. Binary Tree Properties
Each node at most 2 children.

## E. Traversals
- **DFS**: Preorder (Root,Left,Right), Inorder (Left,Root,Right), Postorder (Left,Right,Root)
- **BFS**: Level order using Queue

## F. Recursive vs Iterative
Recursive easy but stack; iterative uses explicit stack/queue.

## G. Common Concepts
Height/Depth, Diameter, LCA, Views (left/right/top/bottom), Balanced check, Path sum, Subtree.

## H. Node
`java
static class TreeNode {
  int val;
  TreeNode left, right;
  TreeNode(int val){ this.val=val; }
}
`

## I. Interview Q&A
Q: Tree vs Graph?
A: Tree is connected acyclic graph, usually has root, no cycles.

Q: Why inorder of BST sorted?
A: Left < Root < Right property.

## J. Revision
- Hierarchical
- Rooted
- Traversals
- DFS/BFS
