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

## Theoretical Analysis

The Capacity Scaling algorithm improves upon the basic Ford-Fulkerson method in several ways:

1. **Reduced number of augmentations**: By focusing on high-capacity paths first, the algorithm reduces the total number of augmentations needed. This is particularly beneficial for networks with a wide range of edge capacities.

2. **Guaranteed polynomial time complexity**: Unlike the basic Ford-Fulkerson, which can have exponential time complexity in certain cases, Capacity Scaling always terminates in polynomial time.

3. **Efficient handling of high-capacity networks**: The algorithm is particularly effective on networks where edge capacities vary significantly, as it quickly pushes large amounts of flow in the early phases.

4. **Gradual refinement**: The phased approach allows for a gradual refinement of the flow, potentially leading to better intermediate solutions if the algorithm is terminated early.

The time complexity of the Capacity Scaling algorithm is O(E^2 log U), where E is the number of edges and U is the maximum edge capacity in the network. This is derived as follows:

- There are at most log U phases (as Δ starts at the largest power of 2 ≤ U and halves each phase).
- In each phase, we perform at most E augmentations (as each augmentation saturates at least one edge in the Δ-residual graph).
- Each augmentation takes O(E) time to find a path and augment the flow.

Therefore, the total time complexity is O(log U * E * E) = O(E^2 log U).

This improvement over the basic Ford-Fulkerson algorithm (which can take O(E * max_flow) time) is significant, especially for networks with large capacities.

## Detailed Example

Let's consider the following network:

```
       16
   A ------> B
   | \       |
12 |  \ 20   | 12
   |   \     |
   v    v    v
   C     D   E
    \   / \ /
   10 \ /   \ 8
      v      v
      F      G
       \    /
        \  / 20
         v
        Sink
```

We'll walk through the Capacity Scaling algorithm step by step:

1. Initial setup:
   - Max capacity = 20
   - Initial Δ = 16 (largest power of 2 not exceeding 20)
   - All flows are 0

2. Phase 1 (Δ = 16):
   - Augmenting path: A -> B -> E -> G -> Sink
   - Bottleneck capacity: 12
   - Flow after augmentation: 12

3. Phase 2 (Δ = 8):
   - Augmenting path: A -> D -> G -> Sink
   - Bottleneck capacity: 8
   - Flow after augmentation: 20

4. Phase 3 (Δ = 4):
   - Augmenting path: A -> C -> F -> Sink
   - Bottleneck capacity: 10
   - Flow after augmentation: 30

5. Phase 4 (Δ = 2):
   - No augmenting paths with capacity ≥ 2

6. Phase 5 (Δ = 1):
   - No augmenting paths with capacity ≥ 1

The algorithm terminates with a maximum flow of 30.

## Comparison with Basic Ford-Fulkerson

Let's compare the Capacity Scaling algorithm with the basic Ford-Fulkerson algorithm:

| Aspect | Ford-Fulkerson | Capacity Scaling |
|--------|----------------|-------------------|
| Time Complexity | O(E * max_flow) | O(E^2 log U) |
| Performance on High-Capacity Networks | Can be slow due to many small augmentations | Efficient, focuses on high-capacity paths first |
| Guaranteed Termination | Not guaranteed for irrational capacities | Always terminates in polynomial time |
| Number of Augmentations | Can be high, especially for networks with widely varying capacities | Generally lower, especially for networks with widely varying capacities |
| Implementation Complexity | Simpler | More complex, requires maintaining Δ-residual graph |
| Intermediate Results | Can have many small improvements | Tends to have larger improvements in early phases |
| Sensitivity to Path Selection | Highly sensitive | Less sensitive due to capacity thresholding |

## Advantages of Capacity Scaling

1. **Efficiency on High-Capacity Networks**: Particularly well-suited for networks with a wide range of capacities, which is common in many real-world scenarios including financial networks.

2. **Predictable Performance**: The polynomial time complexity provides a more predictable runtime compared to basic Ford-Fulkerson.

3. **Potential for Early Termination**: In scenarios where an approximate solution is acceptable, the algorithm can be terminated early while still providing a reasonable flow value.

4. **Reduced Path Finding Operations**: By considering only high-capacity edges in early phases, the algorithm often reduces the number of path-finding operations required.

## Considerations for Implementation

1. **Memory Usage**: Maintaining the Δ-residual graph may require additional memory compared to basic Ford-Fulkerson.

2. **Complexity**: The implementation is more complex, requiring careful management of the scaling factor and the Δ-residual graph.

3. **Integer Capacities**: The algorithm assumes integer capacities. For networks with floating-point capacities, appropriate scaling and rounding strategies need to be employed.

4. **Initial Δ Selection**: The efficiency of the algorithm can be sensitive to the initial choice of Δ. In practice, heuristics other than the largest power of 2 might be more effective for certain network structures.

In our project, implementing both the basic Ford-Fulkerson and the Capacity Scaling algorithms allows us to compare their performance in different scenarios and choose the most appropriate algorithm based on the characteristics of the financial network being analyzed. The Capacity Scaling algorithm is particularly valuable for our use case, as financial networks often involve transactions with widely varying values, mirroring the high-capacity edges that this algorithm handles efficiently.