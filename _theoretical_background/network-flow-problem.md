---
layout: default
title: Network Flow Problem
parent: Theoretical Background
nav_order: 1
---

# Network Flow Problem

The network flow problem is a fundamental concept in graph theory and optimization with numerous real-world applications. It involves finding the maximum flow that can be routed through a network while respecting capacity constraints on the edges.

## Basic Concepts

1. **Network**: A directed graph G = (V, E), where V is the set of vertices (nodes) and E is the set of edges.

2. **Source (s)**: A special node where flow originates.

3. **Sink (t)**: A special node where flow terminates.

4. **Capacity**: Each edge (u, v) has a non-negative capacity c(u, v) ≥ 0, representing the maximum amount of flow that can pass through that edge.

5. **Flow**: A function f: E → R+ that assigns a non-negative real number to each edge, representing the amount of flow passing through that edge.

## Mathematical Formulation

Let G = (V, E) be a directed graph with a source s and sink t. For each edge (u, v) ∈ E, let c(u, v) denote its capacity and f(u, v) the flow through it.

The maximum flow problem can be formulated as:

Maximize: Σ f(s, v) for all v ∈ V
Subject to:

1. Capacity constraint: 0 ≤ f(u, v) ≤ c(u, v) for all (u, v) ∈ E
2. Flow conservation: Σ f(u, v) = Σ f(v, w) for all v ∈ V - {s, t}
3. Skew symmetry: f(u, v) = -f(v, u) for all (u, v) ∈ E

## Constraints

1. **Capacity Constraint**: For each edge (u, v), the flow must not exceed its capacity: 0 ≤ f(u, v) ≤ c(u, v)

2. **Flow Conservation**: For each node v (except source and sink), the sum of incoming flow must equal the sum of outgoing flow:
   Σ f(u, v) = Σ f(v, w) for all v ∈ V - {s, t}

3. **Skew Symmetry**: The flow from u to v must be the opposite of the flow from v to u: f(u, v) = -f(v, u)

## Variations of Network Flow Problems

1. **Minimum Cost Flow**: Find the maximum flow with the minimum total cost, where each edge has an associated cost per unit of flow.

2. **Multi-Commodity Flow**: Multiple commodities need to be routed through the network, each with its own source and sink.

3. **Circulation Problem**: No explicit source or sink; instead, each node has a demand or supply.

4. **Maximum Bipartite Matching**: A special case where the network represents a bipartite graph, and the goal is to find the maximum number of matched pairs.

## Applications

1. **Transportation Networks**: 
   Example: Optimizing the flow of vehicles through a city's road network during rush hour. Nodes represent intersections, edges represent road segments, and capacities represent the maximum number of vehicles each road can handle per hour.

2. **Computer Networks**: 
   Example: Managing data flow in a content delivery network (CDN). Nodes represent servers, edges represent network connections, and capacities represent bandwidth limits.

3. **Financial Systems**: 
   Example: Modeling and optimizing cash flow in a supply chain finance network. Nodes represent companies, edges represent financial relationships, and capacities represent credit limits or available funds.

4. **Bipartite Matching**: 
   Example: Assigning tasks to workers in a project management system. Nodes represent tasks and workers, edges represent possible assignments, and the goal is to maximize the number of tasks assigned while respecting worker capabilities.

5. **Image Segmentation**: 
   Example: In medical imaging, separating tumor regions from healthy tissue. Nodes represent pixels, edges represent connections between neighboring pixels, and capacities are based on color or intensity differences.

6. **Supply Chain Management**: 
   Example: Optimizing the distribution of products from factories to warehouses to retail stores. Nodes represent locations, edges represent transportation routes, and capacities represent shipping or storage limits.

In our project, we apply network flow algorithms to optimize transactions in a decentralized financial system. Here, nodes represent accounts, edges represent trust relationships or possible transaction paths, and capacities represent transaction limits or available balances.

## Detailed Example: Decentralized Financial Network

Consider a small decentralized financial network with 5 accounts (A, B, C, D, E) and a specific token T:

```
       10T
   A -------> B
   | \        |
5T |  \ 7T    | 8T
   |   \      |
   v    v     v
   C     D    E
    \   /
   3T \ / 4T
      v
      Sink
```

In this network:
- A is the source (initial holder of tokens)
- There's a virtual sink representing the final destination of tokens
- Edge labels represent capacities (maximum transferable amount)
- The goal is to find the maximum amount of token T that can be transferred from A to the sink

Step-by-step solution:
1. Path A -> B -> E -> Sink: 8T flow
2. Path A -> C -> Sink: 3T flow
3. Path A -> D -> Sink: 4T flow

Total maximum flow: 15T

This example demonstrates how the network flow problem can model complex financial transactions, accounting for trust relationships (represented by edges) and balance limits (represented by capacities).

Understanding the network flow problem is crucial for grasping the algorithms and implementations discussed in subsequent sections of this documentation. The ability to model real-world scenarios as network flow problems is a powerful tool in optimization and decision-making across various domains.