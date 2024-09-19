# Capacity Scaling Algorithm

The Capacity Scaling algorithm is an enhanced version of the Ford-Fulkerson method for solving the maximum flow problem. It addresses some of the inefficiencies of the basic Ford-Fulkerson algorithm by focusing on pushing flow along edges with large residual capacities first.

## Motivation

The basic Ford-Fulkerson algorithm may spend a lot of time augmenting flow along paths with small residual capacities, leading to many iterations and poor performance on networks with high-capacity edges. Capacity Scaling aims to reduce the number of augmentations by prioritizing high-capacity paths.

## Key Concepts

1. **Scaling Factor (Δ)**: A threshold value used to determine which edges to consider in each phase of the algorithm.

2. **Δ-residual graph**: A subgraph of the residual graph containing only edges with residual capacity ≥ Δ.

3. **Phases**: The algorithm proceeds in phases, with each phase using a different scaling factor.

## Algorithm Steps

1. Initialize flow to 0 for all edges.
2. Set the initial scaling factor Δ to the largest power of 2 not exceeding the maximum edge capacity in the network.
3. While Δ ≥ 1:
   a. While there exists an augmenting path in the Δ-residual graph:
      i. Find an augmenting path in the Δ-residual graph.
      ii. Compute the bottleneck capacity along this path.
      iii. Augment the flow along the path.
      iv. Update the residual graph.
   b. Δ = Δ / 2 (reduce the scaling factor for the next phase)
4. Return the total flow.

## Pseudocode

```
function capacity_scaling(graph, source, sink):
    flow = 0
    Δ = largest_power_of_2(max_edge_capacity(graph))
    while Δ ≥ 1:
        while true:
            path = find_augmenting_path_delta(graph, source, sink, Δ)
            if path is empty:
                break
            bottleneck = min(residual_capacity(e) for e in path)
            for edge in path:
                augment_flow(edge, bottleneck)
            flow += bottleneck
        Δ = Δ / 2
    return flow
```

## Time Complexity

The time complexity of the Capacity Scaling algorithm is O(E^2 log U), where E is the number of edges and U is the maximum edge capacity in the network. This is a significant improvement over the basic Ford-Fulkerson algorithm, especially for networks with high-capacity edges.

## Advantages

1. **Fewer augmentations**: By focusing on high-capacity paths first, the algorithm reduces the total number of augmentations needed.

2. **Better performance on high-capacity networks**: The algorithm performs particularly well on networks where edge capacities vary significantly.

3. **Polynomial time complexity**: Unlike the basic Ford-Fulkerson, Capacity Scaling guarantees termination in polynomial time.

## Considerations

1. **Implementation complexity**: The algorithm is more complex to implement than the basic Ford-Fulkerson method.

2. **Memory usage**: Maintaining the Δ-residual graph may require additional memory.

3. **Performance on low-capacity networks**: For networks with uniformly low capacities, the advantage over simpler algorithms may be less pronounced.

In our project, we implement both the basic Ford-Fulkerson and the Capacity Scaling algorithm, allowing for comparison of their performance in different scenarios.