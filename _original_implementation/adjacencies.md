---
layout: default
title: Adjacencies
parent: Original Implementation
nav_order: 2
---

# Original Implementation: Adjacencies

The `Adjacencies` structure in the original implementation was designed to efficiently manage and query the connectivity of the graph. It employed a lazy evaluation strategy to potentially improve performance, especially for large, sparse graphs.

## Structure

```rust
struct Adjacencies<'a> {
    edges: &'a EdgeDB,
    lazy_adjacencies: HashMap<Node, HashMap<Node, U256>>,
    capacity_adjustments: HashMap<Node, HashMap<Node, U256>>,
}
```

- `edges`: A reference to the `EdgeDB` containing the graph's edge information.
- `lazy_adjacencies`: A cache of computed adjacencies, populated on-demand.
- `capacity_adjustments`: Stores temporary adjustments to edge capacities during flow computation.

## Key Features

### 1. Lazy Evaluation

The `Adjacencies` structure used lazy evaluation to compute and cache adjacency information only when needed. This was implemented in the `adjacencies_from` method:

```rust
impl<'a> Adjacencies<'a> {
    fn adjacencies_from(&mut self, from: &Node) -> HashMap<Node, U256> {
        self.lazy_adjacencies
            .entry(from.clone())
            .or_insert_with(|| {
                let mut result: HashMap<Node, U256> = HashMap::new();
                match from {
                    Node::Node(from) => {
                        for edge in self.edges.outgoing(from) {
                            result
                                .entry(balance_node(edge))
                                .and_modify(|c| {
                                    if edge.capacity > *c {
                                        *c = edge.capacity;
                                    }
                                })
                                .or_insert(edge.capacity);
                        }
                    }
                    Node::BalanceNode(from, token) => {
                        for edge in self.edges.outgoing(from) {
                            if edge.from == *from && edge.token == *token {
                                result.insert(trust_node(edge), edge.capacity);
                            }
                        }
                    }
                    Node::TrustNode(to, token) => {
                        let is_return_to_owner = *to == *token;
                        let mut capacity = U256::from(0);
                        for edge in self.edges.incoming(to) {
                            if edge.token == *token {
                                if is_return_to_owner {
                                    capacity += edge.capacity
                                } else {
                                    capacity = max(capacity, edge.capacity)
                                }
                            }
                            result.insert(Node::Node(*to), capacity);
                        }
                    }
                }
                result
            })
            .clone()
    }
}
```

This approach potentially saved computation time and memory for nodes that were not accessed during the flow computation.

### 2. Capacity Adjustments

The structure allowed for temporary adjustments to edge capacities without modifying the underlying `EdgeDB`:

```rust
impl<'a> Adjacencies<'a> {
    pub fn adjust_capacity(&mut self, from: &Node, to: &Node, adjustment: U256) {
        *self
            .capacity_adjustments
            .entry(from.clone())
            .or_default()
            .entry(to.clone())
            .or_default() += adjustment;
    }
}
```

This feature was crucial for implementing the residual graph concept in the Ford-Fulkerson algorithm without actually modifying the original graph structure.

### 3. Efficient Querying

The `outgoing_edges_sorted_by_capacity` method provided an efficient way to retrieve and sort the outgoing edges of a node:

```rust
impl<'a> Adjacencies<'a> {
    pub fn outgoing_edges_sorted_by_capacity(&mut self, from: &Node) -> Vec<(Node, U256)> {
        let mut adjacencies = self.adjacencies_from(from);
        if let Some(adjustments) = self.capacity_adjustments.get(from) {
            for (node, c) in adjustments {
                *adjacencies.entry(node.clone()).or_default() += *c;
            }
        }
        let mut result = adjacencies
            .into_iter()
            .filter(|(_, cap)| *cap != U256::from(0))
            .collect::<Vec<(Node, U256)>>();
        result.sort_unstable_by_key(|(addr, capacity)| (Reverse(*capacity), addr.clone()));
        result
    }
}
```

This method was particularly useful in the path finding stage of the flow algorithm, allowing for quick identification of high-capacity paths.

## Lazy Evaluation Strategy

The lazy evaluation strategy employed in the `Adjacencies` structure had several key aspects:

1. **On-Demand Computation**: Adjacencies for a node were only computed when explicitly requested through the `adjacencies_from` method.

2. **Caching**: Once computed, adjacencies were stored in the `lazy_adjacencies` HashMap, avoiding redundant computations for frequently accessed nodes.

3. **Immutable Base Graph**: The strategy allowed for working with an immutable `EdgeDB`, as all modifications were handled through the `capacity_adjustments` HashMap.

4. **Memory Efficiency**: For large, sparse graphs, this approach could significantly reduce memory usage by only storing computed adjacencies.

## Performance Analysis

The performance characteristics of the `Adjacencies` structure can be analyzed as follows:

1. **Time Complexity**:
   - First access to a node's adjacencies: O(E), where E is the number of edges connected to the node.
   - Subsequent accesses: O(1) for retrieval from the cache.
   - Capacity adjustments: O(1) for each adjustment.
   - Sorting outgoing edges: O(E log E) where E is the number of outgoing edges for a node.

2. **Space Complexity**:
   - Worst case: O(V^2) where V is the number of nodes, if all adjacencies are computed and stored.
   - Best case: O(1) if only a few nodes' adjacencies are ever accessed.

3. **Trade-offs**:
   - The lazy evaluation strategy trades off some initial computation time for potential memory savings.
   - For dense graphs or algorithms that access most nodes, the strategy may not provide significant benefits and could introduce overhead.

4. **Scenario Analysis**:
   - Best Case: Sparse graph with localized flow computations. Many nodes' adjacencies are never computed, saving memory and computation time.
   - Worst Case: Dense graph with flow computations that access most nodes. All adjacencies end up being computed and stored, potentially using more memory than a direct adjacency list approach.

## Limitations

1. **Memory Usage**: For dense graphs, caching all adjacencies could potentially use a significant amount of memory.

2. **Complexity**: The lazy evaluation and capacity adjustment mechanisms added complexity to the code, potentially making it harder to understand and maintain.

3. **Lack of Incoming Edge Support**: The structure was primarily designed for querying outgoing edges, which could limit its usefulness in bidirectional search algorithms.

4. **Performance in Dynamic Graphs**: If the underlying graph structure changed frequently, the cached adjacencies could become outdated, requiring frequent recomputation.

## Conclusion

The `Adjacencies` structure in the old implementation provided an efficient way to manage graph connectivity with features like lazy evaluation and temporary capacity adjustments. Its design was particularly suited for sparse graphs and algorithms that don't need to access all nodes.

However, its complexity and potential memory usage in certain scenarios were factors that influenced the design of the new implementation. The lessons learned from this approach, particularly the benefits of lazy evaluation and the need for efficient capacity adjustments, informed the development of the new graph structure, which aimed to balance these concerns with greater flexibility and scalability.