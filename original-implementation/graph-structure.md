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

This structure allowed for a clear distinction between different types of nodes in the network, enabling the algorithm to handle balance and trust relationships separately.

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

## EdgeDB

The `EdgeDB` struct was the primary data structure for storing and querying the graph:

```rust
struct EdgeDB {
    edges: HashMap<Address, Vec<Edge>>,
    incoming: HashMap<Address, Vec<Edge>>,
}
```

### Key Features of EdgeDB

1. **Efficient Edge Storage**: Edges were stored in hashmaps, allowing for quick access to outgoing and incoming edges for any node.

2. **Query Methods**:
   - `outgoing(&self, from: &Address) -> impl Iterator<Item = &Edge>`
   - `incoming(&self, to: &Address) -> impl Iterator<Item = &Edge>`

   These methods provided efficient ways to iterate over outgoing and incoming edges for any node.

3. **Edge Updates**: The `update(&mut self, edge: Edge)` method allowed for updating or adding new edges to the graph.

4. **Edge Count**: The `edge_count(&self) -> usize` method returned the total number of edges in the graph.

## Limitations

1. **Flexibility**: The rigid structure of `Node` and `Edge` made it difficult to adapt the implementation for different types of network flow problems.

2. **Scalability**: For very large graphs, storing all edges in memory could potentially lead to performance issues.

3. **Limited Query Capabilities**: While efficient for basic operations, the structure lacked support for more complex graph queries that might be useful in advanced flow algorithms.

4. **No Built-in Support for Residual Graph**: The implementation required manual handling of residual capacities during flow computation.

## Conclusion

The graph structure in the old implementation provided a solid foundation for representing financial networks and computing flows. However, its specialized nature and certain limitations in flexibility and scalability led to the development of a new, more generalized implementation that we'll explore in the next sections.