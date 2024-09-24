---
layout: default
title: Path Search
parent: Original Implementation
nav_order: 2
---

# New Implementation: Path Search

The new implementation introduces a flexible and modular approach to path searching, which is crucial for the performance of network flow algorithms. This design allows for easy integration of different search strategies and comparison of their performance in various network scenarios.

## PathSearchStrategy Trait

The core of the new path search implementation is the `PathSearchStrategy` trait:

```rust
pub trait PathSearchStrategy {
    fn find_path(
        &self,
        graph: &mut FlowGraph,
        source: &Address,
        sink: &Address,
        requested_flow: U256,
        max_distance: Option<u64>,
        scale: Option<U256>,
    ) -> (U256, Vec<Address>);
}
```

This trait defines a common interface for all path search algorithms, allowing them to be easily swapped or compared.

## Implemented Search Strategies

### 1. Breadth-First Search (BFS)

```rust
pub struct BFSPathSearch;

impl PathSearchStrategy for BFSPathSearch {
    fn find_path(
        &self,
        graph: &mut FlowGraph,
        source: &Address,
        sink: &Address,
        requested_flow: U256,
        max_distance: Option<u64>,
        scale: Option<U256>,
    ) -> (U256, Vec<Address>) {
        // BFS implementation...
    }
}
```

BFS finds the shortest path in terms of the number of edges, which can be beneficial for certain network structures.

### 2. Bidirectional BFS

```rust
pub struct BiBFSPathSearch;

impl PathSearchStrategy for BiBFSPathSearch {
    fn find_path(
        &self,
        graph: &mut FlowGraph,
        source: &Address,
        sink: &Address,
        requested_flow: U256,
        max_distance: Option<u64>,
        scale: Option<U256>,
    ) -> (U256, Vec<Address>) {
        // Bidirectional BFS implementation...
    }
}
```

Bidirectional BFS can be more efficient than standard BFS, especially in large networks where the source and sink are far apart.

## Path Search Selection

The implementation allows for dynamic selection of the path search algorithm:

```rust
pub enum PathSearchAlgorithm {
    BFS,
    BiBFS,
}

pub struct PathSearch;

impl PathSearch {
    pub fn find_path(
        algorithm: PathSearchAlgorithm,
        graph: &mut FlowGraph,
        source: &Address,
        sink: &Address,
        requested_flow: U256,
        max_distance: Option<u64>,
        scale: Option<U256>,
    ) -> (U256, Vec<Address>) {
        let strategy: Box<dyn PathSearchStrategy> = match algorithm {
            PathSearchAlgorithm::BFS => Box::new(BFSPathSearch),
            PathSearchAlgorithm::BiBFS => Box::new(BiBFSPathSearch),
        };

        strategy.find_path(graph, source, sink, requested_flow, max_distance, scale)
    }
}
```

This design allows for easy addition of new search strategies in the future.

## Key Features

1. **Flexibility**: Easy to add or modify search strategies without affecting the rest of the system.
2. **Performance Optimization**: Ability to choose the best search strategy for different network structures.
3. **Consistency**: All search strategies adhere to the same interface, ensuring consistent integration with flow algorithms.
4. **Parameterization**: Support for maximum distance and scale parameters allows for fine-tuning of the search process.

## Improvements Over the Old Implementation

1. **Multiple Search Strategies**: Unlike the old implementation, which used a single search method, the new design supports multiple strategies.
2. **Modularity**: Clear separation between the search algorithm and the flow computation logic.
3. **Extensibility**: Easy to add new search strategies as needed.
4. **Performance**: Ability to choose the most efficient search strategy for a given network structure.

## Considerations

1. **Algorithm Selection**: The choice of search strategy can significantly impact performance, requiring careful consideration based on the network characteristics.
2. **Memory Usage**: Some search strategies (e.g., Bidirectional BFS) may use more memory than others.
3. **Completeness**: Ensure that all implemented search strategies are complete (i.e., will find a path if one exists) to maintain the correctness of the flow algorithms.

## Conclusion

The new path search implementation provides a flexible and powerful foundation for network flow algorithms. By allowing different search strategies to be easily integrated and compared, it enables better performance optimization and adaptability to various network structures. This modular design sets the stage for more efficient and versatile flow computations, which we'll explore in the subsequent sections on Ford-Fulkerson and Capacity Scaling implementations.