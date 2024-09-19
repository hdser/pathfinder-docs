# Network Flow Problem

The network flow problem is a fundamental concept in graph theory and optimization with numerous real-world applications. It involves finding the maximum flow that can be routed through a network while respecting capacity constraints on the edges.

## Basic Concepts

1. **Network**: A directed graph G = (V, E), where V is the set of vertices (nodes) and E is the set of edges.

2. **Source (s)**: A special node where flow originates.

3. **Sink (t)**: A special node where flow terminates.

4. **Capacity**: Each edge (u, v) has a non-negative capacity c(u, v) ≥ 0, representing the maximum amount of flow that can pass through that edge.

5. **Flow**: A function f: E → R+ that assigns a non-negative real number to each edge, representing the amount of flow passing through that edge.

## Constraints

1. **Capacity Constraint**: For each edge (u, v), the flow must not exceed its capacity: 0 ≤ f(u, v) ≤ c(u, v)

2. **Flow Conservation**: For each node v (except source and sink), the sum of incoming flow must equal the sum of outgoing flow:
   Σ f(u, v) = Σ f(v, w) for all v ∈ V - {s, t}

3. **Skew Symmetry**: The flow from u to v must be the opposite of the flow from v to u: f(u, v) = -f(v, u)

## Objective

The goal is to maximize the total flow from the source to the sink while respecting all constraints.

## Applications

1. **Transportation Networks**: Optimizing the flow of vehicles, goods, or information through a transportation system.

2. **Computer Networks**: Managing data flow in communication networks.

3. **Financial Systems**: Modeling and optimizing cash flow in financial networks.

4. **Bipartite Matching**: Solving assignment problems in various domains.

5. **Image Segmentation**: In computer vision, for separating foreground from background.

6. **Supply Chain Management**: Optimizing the flow of goods from suppliers to consumers.

In our project, we apply network flow algorithms to optimize transactions in a decentralized financial system, where nodes represent accounts, edges represent trust relationships or possible transaction paths, and capacities represent transaction limits or available balances.

Understanding the network flow problem is crucial for grasping the algorithms and implementations discussed in subsequent sections of this documentation.