---
layout: default
title: Network Flow Problem
parent: Theoretical Background
nav_order: 2
---

# Ford-Fulkerson Algorithm

The Ford-Fulkerson algorithm, developed by L. R. Ford Jr. and D. R. Fulkerson in 1956, is a fundamental method for solving the maximum flow problem in a flow network. This algorithm forms the basis for many network flow algorithms and provides important insights into the nature of network flows.

## Basic Principle

The core idea of the Ford-Fulkerson algorithm is to repeatedly find augmenting paths from the source to the sink in the residual graph and push flow along these paths until no more augmenting paths exist.

## Key Concepts

1. **Augmenting Path**: A path from the source to the sink in the residual graph along which more flow can be pushed.

2. **Residual Graph**: A graph that represents the remaining capacity on each edge after some flow has been pushed through the network. It includes both forward edges (with remaining capacity) and backward edges (allowing flow cancellation).

3. **Residual Capacity**: The additional amount of flow that can be pushed along an edge in the residual graph. For forward edges, it's the unused capacity; for backward edges, it's the amount of flow that can be cancelled.

## Algorithm Steps

1. Initialize flow to 0 for all edges.
2. While there exists an augmenting path from source to sink in the residual graph:
   a. Find an augmenting path (e.g., using DFS or BFS).
   b. Compute the bottleneck capacity (minimum residual capacity along the path).
   c. Augment the flow by adding the bottleneck capacity to the flow of each edge in the path.
   d. Update the residual graph.
3. Return the total flow.

## Pseudocode

```
function ford_fulkerson(graph, source, sink):
    initialize flow to 0 for all edges
    while there exists an augmenting path p from source to sink in the residual graph:
        let bottleneck = min(residual_capacity(e) for e in p)
        for each edge (u, v) in p:
            if (u, v) is a forward edge:
                increase flow on (u, v) by bottleneck
            else:
                decrease flow on (v, u) by bottleneck
    return sum of flow into sink
```

## Correctness Proof

The correctness of the Ford-Fulkerson algorithm is based on the Max-Flow Min-Cut theorem. Here's an outline of the proof:

1. **Termination**: The algorithm terminates because each augmentation increases the total flow by at least the minimum capacity of any edge (assuming integer capacities).

2. **Optimality**: When the algorithm terminates, there are no augmenting paths in the residual graph. This implies that the set of nodes reachable from the source in the residual graph forms a minimum cut in the original graph.

3. **Max-Flow Min-Cut Theorem**: The value of the maximum flow is equal to the capacity of the minimum cut. Since we've found a flow equal to a cut capacity, this flow must be maximum.

## Detailed Example

Consider the following network:

```
      10
  A ------> B
  | \       |
6 |  \ 8    | 9
  |   \     |
  v    v    v
  C     D   E
   \    |  /
  4 \   | / 7
     \  | /
      v v v
       Sink
```

Step-by-step execution:

1. Initial flow: 0
   Residual graph: Same as original graph

2. Find augmenting path: A -> B -> E -> Sink
   Bottleneck: 9
   New flow: 9
   Residual graph: Update capacities

3. Find augmenting path: A -> C -> Sink
   Bottleneck: 4
   New flow: 13
   Residual graph: Update capacities

4. Find augmenting path: A -> D -> Sink
   Bottleneck: 7
   New flow: 20
   Residual graph: Update capacities

5. No more augmenting paths
   Maximum flow: 20

## Time Complexity

The time complexity of the Ford-Fulkerson algorithm depends on how the augmenting paths are found and the capacities of the edges:

- Worst-case: O(E * max_flow), where E is the number of edges and max_flow is the maximum flow value.
- Using Edmonds-Karp algorithm (which uses BFS to find augmenting paths): O(V * E^2), where V is the number of vertices.

## Limitations

1. **Non-polynomial time complexity**: In networks with irrational capacities, the algorithm may not terminate.
2. **Sensitivity to augmenting path selection**: The efficiency can vary greatly depending on how augmenting paths are chosen.

## Improvements and Variations

1. **Edmonds-Karp Algorithm**: 
   - Uses BFS to find augmenting paths
   - Guarantees polynomial time complexity: O(VE^2)
   - Always chooses the shortest augmenting path

2. **Dinic's Algorithm**: 
   - Uses level graphs to find blocking flows
   - Improves time complexity to O(V^2E)
   - Particularly efficient for unit capacity networks

3. **Push-Relabel Algorithm**: 
   - A different approach that maintains a preflow instead of a valid flow
   - Works locally on nodes, pushing excess flow and relabeling heights
   - Achieves O(V^2E) time complexity, with better practical performance

4. **Capacity Scaling**: 
   - Modification of Ford-Fulkerson that considers only high-capacity augmenting paths initially
   - Gradually reduces the capacity threshold
   - Achieves O(E^2 log U) time complexity, where U is the maximum edge capacity

In our project, we implement both the basic Ford-Fulkerson algorithm and an improved version using capacity scaling. This allows us to compare their performance and choose the most appropriate algorithm based on the characteristics of the financial network being analyzed.

Understanding the Ford-Fulkerson algorithm and its variations is crucial for implementing efficient network flow solutions in our decentralized financial system. The choice between these algorithms can significantly impact the performance and scalability of our transaction optimization process.