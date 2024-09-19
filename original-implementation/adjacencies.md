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
fn adjacencies_from(&mut self, from: &Node) -> HashMap<Node, U256> {
    self.lazy_adjacencies
        .entry(from.clone())
        .or_insert_with(|| {
            // Compute adjacencies for the node
            // ...
        })
        .clone()
}
```

This approach potentially saved computation time and memory for nodes that were not accessed during the flow computation.

### 2. Capacity Adjustments

The structure allowed for temporary adjustments to edge capacities without modifying the underlying `EdgeDB`:

```rust
pub fn adjust_capacity(&mut self, from: &Node, to: &Node, adjustment: U256) {
    *self
        .capacity_adjustments
        .entry(from.clone())
        .or_default()
        .entry(to.clone())
        .or_default() += adjustment;
}
```

This feature was crucial for implementing the residual graph concept in the Ford-Fulkerson algorithm without actually modifying the original graph structure.

### 3. Efficient Querying

The `outgoing_edges_sorted_by_capacity` method provided an efficient way to retrieve and sort the outgoing edges of a node:

```rust
pub fn outgoing_edges_sorted_by_capacity(&mut self, from: &Node) -> Vec<(Node, U256)> {
    let mut adjacencies = self.adjacencies_from(from);
    // Apply capacity adjustments and sort
    // ...
}
```

This method was particularly useful in the path finding stage of the flow algorithm, allowing for quick identification of high-capacity paths.

## Limitations

1. **Memory Usage**: For dense graphs, caching all adjacencies could potentially use a significant amount of memory.

2. **Complexity**: The lazy evaluation and capacity adjustment mechanisms added complexity to the code, potentially making it harder to understand and maintain.

3. **Lack of Incoming Edge Support**: The structure was primarily designed for querying outgoing edges, which could limit its usefulness in bidirectional search algorithms.

4. **Performance in Dynamic Graphs**: If the underlying graph structure changed frequently, the cached adjacencies could become outdated, requiring frequent recomputation.

## Conclusion

The `Adjacencies` structure in the old implementation provided an efficient way to manage graph connectivity with features like lazy evaluation and temporary capacity adjustments. However, its complexity and potential memory usage in certain scenarios were factors that influenced the design of the new implementation, which we'll explore in subsequent sections.