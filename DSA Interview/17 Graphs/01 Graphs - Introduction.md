---
topic: graphs-intro
tags: [dsa, graphs]
status: complete
---
# Graphs - Introduction

## A. Introduction
Graph G(V,E) = set of vertices and edges. Models relationships/networks.

## B. Terminology
Vertex/Node, Edge, Directed/Undirected, Weighted/Unweighted, Adjacent, Degree (in/out), Path, Cycle, Connected, Component, DAG.

## C. Representations
- **Adjacency Matrix**: 2D array, good for dense, O(V^2) space
- **Adjacency List**: Array of lists, good for sparse, O(V+E)
- **Edge List**: list of pairs

## D. Traversals
- **BFS**: uses Queue, shortest path in unweighted
- **DFS**: uses Stack/recursion, explore all

## E. Key Concepts
Connected components, Cycle detection (directed/undirected), Bipartiteness, Topological sort, Shortest paths, MST.

## F. Java Example (Adj List)
`java
ArrayList<Integer>[] graph = new ArrayList[V];
for(int i=0;i<V;i++) graph[i] = new ArrayList<>();
graph[u].add(v);
`

## G. Interview Q&A
Q: Graph vs Tree?
A: Graph may have cycles, no single root required, can be disconnected/directed.

Q: When BFS vs DFS?
A: BFS for shortest in edges (unweighted), DFS for connectivity/path existence/exploration.

## H. Revision
- V,E
- Directed/Undirected
- Adj list/matrix
- BFS/DFS
- Visited array
