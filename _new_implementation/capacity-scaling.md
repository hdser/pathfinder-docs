---
layout: default
title: Capacity Scaling Algorithm
parent: New Implementation
nav_order: 4
---

# New Implementation: Capacity Scaling Algorithm

The Capacity Scaling algorithm is an enhanced version of the Ford-Fulkerson method that aims to improve performance, especially for networks with high-capacity edges. This implementation builds upon the flexible structure established for the Ford-Fulkerson algorithm, integrating seamlessly with the `FlowGraph` and path search strategies

## Implementation Structure

Like the Ford-Fulkerson implementation, the Capacity Scaling algorithm is implemented as a struct that implements the `NetworkFlowAlgorithm` trait:

```rust
#[derive(Clone)]
pub struct CapacityScaling;

impl NetworkFlowAlgorithm for CapacityScaling {
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

Let's break down the Capacity Scaling implementation step by step:

### 1. Initialization and Scaling Factor Determination

```rust
let mut total_flow = U256::default();
let mut sink_flow = U256::default();
let mut flow_paths = Vec::new();
let mut residual_graph = graph.clone();

let max_capacity = residual_graph.get_max_capacity();
let mut scale = U256::from(1);
while scale * U256::from(2) <= max_capacity {
    scale = scale * U256::from(2);
}
let min_flow_increase = U256::from(100000000000000000); // 0.1 tokens

let estimated_max_flow = graph.estimate_max_flow(source, sink);
let target_flow = std::cmp::min(requested_flow, estimated_max_flow);
```

Here, we initialize our flow variables and create a residual graph. The scaling factor is determined by finding the largest power of 2 not exceeding the maximum edge capacity in the network. We also set a minimum flow increase threshold to avoid spending time on insignificant flow improvements.

### 2. Main Loop

```rust
while scale > U256::from(0) {
    let mut tries = 0;
    loop {
        let (path_flow, path) = PathSearch::find_path(
            path_search_algorithm,
            &mut residual_graph,
            source,
            sink,
            target_flow - sink_flow,
            max_distance,
            Some(std::cmp::max(scale, min_flow_increase))
        );
        
        if path_flow == U256::default() || path_flow < min_flow_increase || tries >= max_tries_per_scale {
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

        tries += 1;
    }
    scale = scale / U256::from(2);
}
```

This is the core of the Capacity Scaling algorithm. We repeatedly find augmenting paths with capacity at least equal to the current scale. After exhausting all such paths or reaching a maximum number of tries, we reduce the scale by half and continue. This process continues until the scale becomes zero.

### 3. Flow Update

The flow update method is similar to the one used in Ford-Fulkerson:

```rust
fn update_flow(graph: &mut FlowGraph, path: &[Address], path_flow: U256, flow_paths: &mut Vec<Edge>) -> bool {
    let mut reached_sink = false;
    
    for window in path.windows(2) {
        let from = window[0];
        let to = window[1];
        let token = graph.get_edge_token(&from, &to).expect("Edge not found");
        
        graph.decrease_edge_capacity(&from, &to, &token, path_flow);
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

### 4. Post-processing

```rust
let actual_sink_flow = flow_paths.iter()
    .filter(|e| e.to == *sink)
    .fold(U256::from(0), |acc, e| acc + e.capacity);

let (final_flow, final_transfers) = NetworkFlow::post_process(actual_sink_flow, flow_paths, requested_flow, source, sink);
```

After the main algorithm completes, we calculate the actual flow to the sink and perform post-processing to ensure we meet the requested flow and optimize the transfer paths.

## Performance Optimizations

1. **Scaling Factor**: The use of a scaling factor allows the algorithm to focus on high-capacity paths first, potentially reducing the number of augmenting path computations.

2. **Minimum Flow Threshold**: The `min_flow_increase` threshold prevents the algorithm from spending time on insignificant flow improvements.

3. **Maximum Tries per Scale**: The `max_tries_per_scale` limit prevents the algorithm from getting stuck in a particular scale if many small improvements are possible.

4. **Efficient Path Finding**: By using the `PathSearchAlgorithm` enum, we can choose the most efficient path-finding algorithm for our specific network structure.

5. **Early Termination**: We break the inner loop as soon as we reach or exceed the requested flow to the sink.

## Comparison with Basic Ford-Fulkerson

1. **Efficiency on High-Capacity Networks**: By focusing on high-capacity paths first, the Capacity Scaling algorithm can find a good approximation of the maximum flow more quickly, especially in networks with a wide range of capacities.

2. **Fewer Augmenting Paths**: The scaling approach typically results in fewer, but more significant, flow augmentations compared to the basic Ford-Fulkerson method.

3. **Better Worst-Case Time Complexity**: While the basic Ford-Fulkerson can have poor performance on certain graph structures, Capacity Scaling provides a more consistent performance across different network types.

4. **Adaptability**: The algorithm adapts its behavior based on the network's capacity distribution, making it effective across a wide range of network types.

## Integration with FlowGraph

The Capacity Scaling algorithm leverages several key features of the `FlowGraph` structure:

1. **Maximum Capacity Retrieval**: The `get_max_capacity` method is used to determine the initial scaling factor.

2. **Edge Capacity Updates**: The `decrease_edge_capacity` and `increase_edge_capacity` methods are used to update the residual graph efficiently.

3. **Flow Estimation**: The `estimate_max_flow` method provides an initial estimate of the maximum possible flow, helping to set a realistic target flow.

## Limitations and Future Improvements

1. **Memory Usage**: Like the Ford-Fulkerson implementation, memory usage can be a concern for very large graphs.

2. **Scaling Factor Initialization**: The current implementation uses the largest power of 2 not exceeding the maximum capacity. Future versions could explore more sophisticated methods for initializing and updating the scaling factor.

3. **Parallelization**: The current implementation is single-threaded. Future versions could explore parallelizing certain aspects of the algorithm, such as searching for multiple augmenting paths simultaneously within each scale.

## Conclusion

The Capacity Scaling implementation provides a powerful enhancement to the basic Ford-Fulkerson algorithm, offering improved performance and adaptability for a wide range of network types. By leveraging the flexible `FlowGraph` structure and modular path search strategies, it maintains the advantages of the new implementation architecture while introducing specialized optimizations for high-capacity networks.

This implementation represents a significant step forward in the project's ability to handle large-scale, complex network flow problems efficiently. Its integration with the existing components demonstrates the flexibility and extensibility of the new implementation's design, setting a strong foundation for future enhancements and optimizations in network flow computations for decentralized financial systems.