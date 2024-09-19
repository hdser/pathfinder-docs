# Original Implementation: Flow Computation

The flow computation in the original implementation was based on the Ford-Fulkerson algorithm, with additional features for flow pruning and transfer simplification. This implementation was tailored to the specific needs of the financial network but had limitations in terms of flexibility and performance for large-scale networks.

## Main Algorithm Structure

The main flow computation function had the following signature:

```rust
pub fn compute_flow(
    source: &Address,
    sink: &Address,
    edges: &EdgeDB,
    requested_flow: U256,
    max_distance: Option<u64>,
    max_transfers: Option<u64>,
) -> (U256, Vec<Edge>)
```

This function implemented the Ford-Fulkerson algorithm with additional steps for pruning and simplification.

## Key Components

### 1. Augmenting Path Finding

The algorithm used a modified breadth-first search to find augmenting paths:

```rust
fn augmenting_path(
    source: &Address,
    sink: &Address,
    adjacencies: &mut Adjacencies,
    max_distance: Option<u64>,
) -> (U256, Vec<Node>)
```

This function returned the flow value and the path found. It respected the `max_distance` parameter to limit the search depth.

### 2. Flow Augmentation

After finding an augmenting path, the algorithm updated the flow:

```rust
for window in parents.windows(2) {
    if let [node, prev] = window {
        adjacencies.adjust_capacity(prev, node, -new_flow);
        adjacencies.adjust_capacity(node, prev, new_flow);
        // Update used_edges...
    }
}
```

This step adjusted the residual capacities and kept track of the used edges.

### 3. Flow Pruning

If the computed flow exceeded the requested flow, a pruning step was performed:

```rust
if flow > requested_flow {
    let still_to_prune = prune_flow(source, sink, flow - requested_flow, &mut used_edges);
    flow = requested_flow + still_to_prune;
}
```

The `prune_flow` function attempted to remove excess flow while maintaining flow conservation.

### 4. Transfer Reduction

If a maximum number of transfers was specified, the algorithm attempted to reduce the number of transfers while minimizing the loss of flow:

```rust
if let Some(max_transfers) = max_transfers {
    let lost = reduce_transfers(max_transfers * 3, &mut used_edges);
    flow -= lost;
}
```

The `reduce_transfers` function iteratively removed the smallest transfers until the desired number was reached or no further reduction was possible.

### 5. Transfer Extraction

After computing the flow, the algorithm extracted the actual transfers from the used edges:

```rust
let transfers = if flow == U256::from(0) {
    vec![]
} else {
    extract_transfers(source, sink, &flow, used_edges)
};
```

The `extract_transfers` function converted the internal flow representation into a series of `Edge` structs representing individual transfers.

### 6. Transfer Simplification

The extracted transfers were then simplified to reduce the number of hops:

```rust
let simplified_transfers = simplify_transfers(transfers);
```

This step combined consecutive transfers where possible, reducing the overall number of transactions needed to achieve the computed flow.

### 7. Transfer Sorting

Finally, the transfers were sorted to ensure they could be executed in a valid order:

```rust
let sorted_transfers = sort_transfers(simplified_transfers);
```

This sorting ensured that each account had received sufficient funds before making outgoing transfers.

## Key Features

1. **Flow Pruning**: Allowed the algorithm to compute the maximum flow and then prune it back to the requested amount, potentially finding more efficient paths for smaller flow values.

2. **Transfer Reduction**: Provided a mechanism to limit the number of transfers, which could be important for practical applications where each transfer incurs a cost.

3. **Transfer Simplification**: Reduced the complexity of the final transfer set by combining transfers where possible, making the result more practical for real-world use.

4. **Valid Ordering**: Ensured that the final set of transfers could be executed in the given order without any account temporarily going into debt.

## Limitations

1. **Efficiency**: The multiple post-processing steps (pruning, reduction, simplification, sorting) could be computationally expensive for large networks or high flow values.

2. **Optimality**: While the algorithm found the maximum flow, the post-processing steps might not always produce the optimal set of transfers for the given constraints.

3. **Flexibility**: The implementation was tightly coupled to the specific graph representation and financial network context, making it difficult to adapt to other types of flow problems.

4. **Scalability**: For very large networks, the algorithm might struggle due to its need to keep all used edges in memory and perform multiple passes over the data.

## Conclusion

The flow computation in the old implementation provided a comprehensive solution for finding and optimizing flows in financial networks. It went beyond the basic Ford-Fulkerson algorithm to address practical concerns like transfer count limitations and simplification of the resulting transfer set.

However, its complexity and potential performance issues with large networks were significant factors in the decision to develop a new implementation. The new version aimed to address these limitations while maintaining the practical features that made the old implementation valuable for financial network flow computations.