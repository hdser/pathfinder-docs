---
layout: default
title: Ford-Fulkerson Algorithm
parent: New Implementation
nav_order: 3
---

# New Implementation: Ford-Fulkerson Algorithm

The new implementation of the Ford-Fulkerson algorithm leverages the improved `FlowGraph` structure and the flexible path search strategies to provide a more efficient and adaptable solution for computing maximum flow in a network.

## Implementation Structure

The Ford-Fulkerson algorithm is implemented as a struct that implements the `NetworkFlowAlgorithm` trait:

```rust
#[derive(Clone)]
pub struct FordFulkerson;

impl NetworkFlowAlgorithm for FordFulkerson {
    fn compute_flow(
        &self,
        graph: &FlowGraph,
        source: &Address,
        sink: &Address,
        requested_flow: U256,
        max_distance: Option<u64>,
        max_transfers: Option<u64>,
        path_search_algorithm: PathSearchAlgorithm,
        recorder: Option<&mut FlowRecorder>
    ) -> (U256, Vec<Edge>) {
        // Implementation...
    }
}
```

## Detailed Algorithm Walkthrough

Let's break down the Ford-Fulkerson implementation step by step:

### 1. Initialization

```rust
let mut total_flow = U256::default();
let mut sink_flow = U256::default();
let mut flow_paths = Vec::new();
let mut residual_graph = graph.clone();

let estimated_max_flow = graph.estimate_max_flow(source, sink);
let target_flow = std::cmp::min(requested_flow, estimated_max_flow);
```

Here, we initialize our flow variables and create a residual graph. We also estimate the maximum possible flow and set our target flow accordingly.

### 2. Main Loop

```rust
loop {
    let (path_flow, path) = PathSearch::find_path(
        path_search_algorithm,
        &mut residual_graph,
        source,
        sink,
        target_flow - sink_flow,
        max_distance,
        Some(U256::from(100000000000000000))
    );
    
    if path_flow == U256::default() || path.is_empty() {
        println!("No more augmenting paths found");
        break;
    }

    let reached_sink = Self::update_flow(&mut residual_graph, &path, path_flow, &mut flow_paths);
    if reached_sink {
        sink_flow += path_flow;
    }
    
    total_flow += path_flow;

    if let Some(ref mut recorder) = recorder {
        recorder.record_step(total_flow, path.clone(), path_flow, residual_graph.clone());
    }

    if sink_flow >= target_flow {
        println!("Reached or exceeded requested flow to sink");
        break;
    }
}
```

This is the core of the Ford-Fulkerson algorithm. We repeatedly find augmenting paths and update the flow until no more paths are found or we reach the target flow.

### 3. Flow Update

```rust
fn update_flow(graph: &mut FlowGraph, path: &[Address], path_flow: U256, flow_paths: &mut Vec<Edge>) -> bool {
    let mut reached_sink = false;
    
    for window in path.windows(2) {
        let from = window[0];
        let to = window[1];
        let token = graph.get_edge_token(&from, &to).expect("Edge not found");
        
        // Decrease capacity in the forward direction
        graph.decrease_edge_capacity(&from, &to, &token, path_flow);
        
        // Increase capacity in the reverse direction
        graph.increase_edge_capacity(&to, &from, &token, path_flow);

        flow_paths.push(Edge {
            from,
            to,
            token,
            capacity: path_flow,
        });

        if to == *path.last().unwrap() {
            reached_sink = true;
        }
    }

    reached_sink
}
```

This method updates the residual graph and records the flow paths.

### 4. Post-processing

```rust
let actual_sink_flow = flow_paths.iter()
    .filter(|e| e.to == *sink)
    .fold(U256::from(0), |acc, e| acc + e.capacity);

let (final_flow, final_transfers) = NetworkFlow::post_process(actual_sink_flow, flow_paths, requested_flow, source, sink);
```

After the main algorithm completes, we calculate the actual flow to the sink and perform post-processing to ensure we meet the requested flow and optimize the transfer paths.

## Integration with FlowGraph

The new implementation makes extensive use of the `FlowGraph` structure:

1. **Edge Capacity Updates**: We use `decrease_edge_capacity` and `increase_edge_capacity` methods to update the residual graph.

2. **Path Finding**: The `PathSearch::find_path` method takes a `FlowGraph` as an argument, allowing it to efficiently query the graph structure.

3. **Flow Estimation**: We use the `estimate_max_flow` method of `FlowGraph` to get an initial estimate of the maximum possible flow.

## Performance Optimizations

1. **Efficient Path Finding**: By using the `PathSearchAlgorithm` enum, we can choose the most efficient path-finding algorithm for our specific network structure.

2. **Minimum Flow Threshold**: We use a minimum flow threshold (100000000000000000 wei, or 0.1 tokens) to avoid spending time on insignificant flow improvements.

```rust
Some(U256::from(100000000000000000))
```

3. **Early Termination**: We break the main loop as soon as we reach or exceed the requested flow to the sink.

4. **Residual Graph**: Instead of modifying the original graph, we work with a residual graph, allowing for efficient backtracking of flow.

5. **Flow Recording**: The optional `FlowRecorder` allows for detailed analysis and visualization of the flow computation process without affecting the core algorithm.

## Comparison with Old Implementation

1. **Flexibility**: The new implementation can easily switch between different path search algorithms, allowing for better performance across different network types.

2. **Scalability**: By leveraging the efficient `FlowGraph` structure, the new implementation can handle larger networks more effectively.

3. **Modularity**: The clear separation between the flow algorithm, graph structure, and path search strategy makes the code more maintainable and extensible.

4. **Performance**: The ability to choose optimal path search strategies and the use of efficient data structures in `FlowGraph` lead to better overall performance.

## Limitations and Future Improvements

1. **Memory Usage**: For very large graphs, memory usage can still be a concern, especially when storing the entire residual graph.

2. **Parallelization**: The current implementation is single-threaded. Future versions could explore parallelizing certain aspects of the algorithm, such as path finding.

3. **Dynamic Flow Updates**: The algorithm currently recomputes the entire flow when changes occur. Future improvements could focus on incrementally updating flows in response to small changes in the network.

## Conclusion

The new Ford-Fulkerson implementation provides a more flexible, efficient, and scalable solution for computing maximum flow in complex networks. By leveraging the improved `FlowGraph` structure and modular path search strategies, it addresses many of the limitations of the old implementation while introducing new capabilities for handling large-scale networks and providing insights into the flow computation process.

This implementation sets a solid foundation for further enhancements and optimizations. Its modular design allows for easy integration of new path search algorithms or flow computation strategies, making it well-suited for adapting to the evolving needs of decentralized financial networks.