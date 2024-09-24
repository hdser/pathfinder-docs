---
layout: default
title: Graph Structure
parent: Original Implementation
nav_order: 1
---

# Original Implementation: Graph Structure

The graph structure in the original implementation was designed to represent a financial network with specific requirements for handling different types of nodes and edges. This custom representation was tailored to the needs of the project but had certain limitations in terms of flexibility and scalability.

## Node Representation

Nodes in the graph were represented using an enum `Node`:

```rust
enum Node {
    Node(Address),
    BalanceNode(Address, Address),
    TrustNode(Address, Address),
}
```

- `Node(Address)`: Represents a regular account in the network.
- `BalanceNode(Address, Address)`: Represents the balance of a specific token for an account.
- `TrustNode(Address, Address)`: Represents the trust relationship between two accounts for a specific token.

### Design Decisions
1. The use of an enum allowed for clear distinction between different node types within the same graph structure.
2. `BalanceNode` and `TrustNode` allowed for modeling complex relationships in the financial network, separating token balances from trust relationships.
3. Using `Address` type (likely an alias for a fixed-size array or a struct representing Ethereum addresses) ensured type safety and proper handling of account identifiers.

## Edge Representation

Edges were represented using a struct `Edge`:

```rust
struct Edge {
    from: Address,
    to: Address,
    token: Address,
    capacity: U256,
}
```

- `from`: The source account of the edge.
- `to`: The destination account of the edge.
- `token`: The token involved in this edge (could be a currency or the account itself for trust relationships).
- `capacity`: The maximum flow capacity of this edge.

### Design Decisions
1. The `Edge` struct provided a clear and concise representation of financial relationships and potential transactions.
2. Using `U256` for capacity allowed for handling large numerical values common in financial transactions.
3. Including the `token` in each edge enabled multi-currency support within the same graph structure.

## EdgeDB

The `EdgeDB` struct was the primary data structure for storing and querying the graph:

```rust
struct EdgeDB {
    edges: HashMap<Address, Vec<Edge>>,
    incoming: HashMap<Address, Vec<Edge>>,
}
```

### Key Features of EdgeDB

1. **Efficient Edge Storage**: 
   Edges were stored in hashmaps, allowing for quick access to outgoing and incoming edges for any node.

   ```rust
   impl EdgeDB {
       pub fn new(edges: Vec<Edge>) -> Self {
           let mut db = EdgeDB {
               edges: HashMap::new(),
               incoming: HashMap::new(),
           };
           for edge in edges {
               db.update(edge);
           }
           db
       }

       pub fn update(&mut self, edge: Edge) {
           self.edges.entry(edge.from).or_default().push(edge.clone());
           self.incoming.entry(edge.to).or_default().push(edge);
       }
   }
   ```

2. **Query Methods**:
   ```rust
   impl EdgeDB {
       pub fn outgoing(&self, from: &Address) -> impl Iterator<Item = &Edge> {
           self.edges.get(from).into_iter().flat_map(|edges| edges.iter())
       }

       pub fn incoming(&self, to: &Address) -> impl Iterator<Item = &Edge> {
           self.incoming.get(to).into_iter().flat_map(|edges| edges.iter())
       }
   }
   ```

   These methods provided efficient ways to iterate over outgoing and incoming edges for any node.

3. **Edge Updates**: 
   The `update` method allowed for updating or adding new edges to the graph.

4. **Edge Count**: 
   ```rust
   impl EdgeDB {
       pub fn edge_count(&self) -> usize {
           self.edges.values().map(|edges| edges.len()).sum()
       }
   }
   ```

   This method returned the total number of edges in the graph.

## Memory Trade-offs

1. **Space Efficiency**:
   - Storing edges in both `edges` and `incoming` hashmaps duplicated edge data, increasing memory usage.
   - This trade-off was made to improve query performance for both outgoing and incoming edges.

2. **Query Performance vs Memory Usage**:
   - The use of hashmaps for `edges` and `incoming` provided O(1) average-case lookup time for edges.
   - This came at the cost of additional memory usage compared to a single list of all edges.

3. **Flexibility vs Memory Overhead**:
   - The `Node` enum allowed for representing different node types within the same structure.
   - This flexibility came with a small memory overhead due to the enum's discriminant and potential padding.

4. **Large Number Handling**:
   - Using `U256` for capacities allowed handling of very large numerical values.
   - This choice increased memory usage compared to smaller integer types but was necessary for accurately representing financial values.

## Limitations

1. **Flexibility**: 
   The rigid structure of `Node` and `Edge` made it difficult to adapt the implementation for different types of network flow problems.

2. **Scalability**: 
   For very large graphs, storing all edges in memory could potentially lead to performance issues.

   ```rust
   // This could become problematic for very large graphs
   let all_edges: Vec<&Edge> = self.edges.values().flat_map(|edges| edges.iter()).collect();
   ```

3. **Limited Query Capabilities**: 
   While efficient for basic operations, the structure lacked support for more complex graph queries that might be useful in advanced flow algorithms.

4. **No Built-in Support for Residual Graph**: 
   The implementation required manual handling of residual capacities during flow computation.

   ```rust
   // Example of manual residual capacity handling
   fn update_residual_graph(graph: &mut EdgeDB, path: &[Address], flow: U256) {
       for window in path.windows(2) {
           let (from, to) = (window[0], window[1]);
           // Decrease forward capacity
           if let Some(edge) = graph.outgoing(&from).find(|e| e.to == to) {
               // This manual update could be error-prone
               let new_capacity = edge.capacity - flow;
               graph.update(Edge { capacity: new_capacity, ..edge.clone() });
           }
           // Increase backward capacity
           // ... similar logic for backward edge
       }
   }
   ```

## Conclusion

The graph structure in the old implementation provided a solid foundation for representing financial networks and computing flows. It offered efficient edge lookup and update operations, which were crucial for the flow algorithms implemented.

However, its specialized nature and certain limitations in flexibility and scalability led to the development of a new, more generalized implementation. The new implementation aimed to address these limitations while maintaining the efficient operations that made the original structure effective for financial network computations.

The lessons learned from this implementation, particularly the trade-offs between memory usage and query performance, informed the design decisions in the new implementation, which we'll explore in the next sections.