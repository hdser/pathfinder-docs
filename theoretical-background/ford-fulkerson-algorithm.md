# Ford-Fulkerson Algorithm

The Ford-Fulkerson algorithm is a fundamental method for solving the maximum flow problem in a flow network. Developed by L. R. Ford Jr. and D. R. Fulkerson in 1956, this algorithm forms the basis for many network flow algorithms.

## Basic Principle

The core idea of the Ford-Fulkerson algorithm is to repeatedly find augmenting paths from the source to the sink in the residual graph and push flow along these paths until no more augmenting paths exist.

## Key Concepts

1. **Augmenting Path**: A path from the source to the sink in the residual graph along which more flow can be pushed.

2. **Residual Graph**: A graph that represents the remaining capacity on each edge after some flow has been pushed through the network.

3. **Residual Capacity**: The additional amount of flow that can be pushed along an edge in the residual graph.

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
    flow = 0
    while true:
        path = find_augmenting_path(graph, source, sink)
        if path is empty:
            break
        bottleneck = min(residual_capacity(e) for e in path)
        for edge in path:
            augment_flow(edge, bottleneck)
        flow += bottleneck
    return flow
```

## Time Complexity

The time complexity of the Ford-Fulkerson algorithm depends on how the augmenting paths are found and the capacities of the edges:

- Worst-case: O(E * max_flow), where E is the number of edges and max_flow is the maximum flow value.
- Using Edmonds-Karp algorithm (which uses BFS to find augmenting paths): O(V * E^2), where V is the number of vertices.

## Limitations

1. **Non-polynomial time complexity**: In networks with irrational capacities, the algorithm may not terminate.
2. **Sensitivity to augmenting path selection**: The efficiency can vary greatly depending on how augmenting paths are chosen.

## Improvements

Several improvements and variations of the Ford-Fulkerson algorithm have been developed to address its limitations:

1. **Edmonds-Karp Algorithm**: Uses BFS to find augmenting paths, guaranteeing polynomial time complexity.
2. **Dinic's Algorithm**: Uses level graphs to find blocking flows, improving time complexity.
3. **Push-Relabel Algorithm**: A different approach that maintains a preflow instead of a valid flow throughout the algorithm.

In our project, we implement both the basic Ford-Fulkerson algorithm and an improved version using capacity scaling, which we'll discuss in the next section.