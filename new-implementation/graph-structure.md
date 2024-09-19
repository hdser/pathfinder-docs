# New Implementation: Graph Structure

The new implementation introduces a more flexible and efficient graph structure called `FlowGraph`. This structure is designed to handle large-scale networks more effectively and provides a cleaner interface for flow algorithms.

## FlowGraph Structure

```rust
pub struct FlowGraph {
    nodes: HashSet<Address>,
    edges: HashMap<EdgeKey, U256>,
    trust_limits: HashMap<(Address, Address), U256>,
    balances: HashMap<(Address, Address), U256>,
    flows: HashMap<EdgeKey, U256>,
    total_flows: HashMap<(Address, Address), U256>,
    outgoing_edges: HashMap<Address, Vec<EdgeKey>>,
    incoming_edges: HashMap<Address, Vec<EdgeKey>>,
    outgoing_edges_cache: RefCell<HashMap<Address, Vec<(Address, Address, U256)>>>,
    incoming_edges_cache: RefCell<HashMap<Address, Vec<(Address, Address, U256)>>>,
    dirty_nodes: RefCell<HashSet<Address>>,
}
```

### Key Components

1. `nodes`: A set of all nodes (addresses) in the graph.
2. `edges`: A map of edge keys to their capacities.
3. `trust_limits`: Stores trust limits between pairs of addresses.
4. `balances`: Stores token balances for each address.
5. `flows` and `total_flows`: Keep track of current flows in the network.
6. `outgoing_edges` and `incoming_edges`: Efficiently store edge connectivity.
7. `outgoing_edges_cache` and `incoming_edges_cache`: Cache computed edge information for performance.
8. `dirty_nodes`: Tracks nodes whose cached information needs updating.

## Key Features

### 1. Efficient Edge Representation

Edges are represented using an `EdgeKey` struct:

```rust
pub struct EdgeKey {
    pub from: Address,
    pub to: Address,
    pub token: Address,
}
```

This allows for efficient storage and retrieval of edge information.

### 2. Dynamic Graph Updates

The structure supports dynamic updates to the graph:

```rust
pub fn add_edge(&mut self, from: Address, to: Address, token: Address, capacity: U256) {
    // Implementation...
}

pub fn remove_edge(&mut self, key: &EdgeKey) {
    // Implementation...
}
```

These methods automatically update all relevant data structures and invalidate caches as necessary.

### 3. Caching with Lazy Evaluation

The `get_outgoing_edges` and `get_incoming_edges` methods use caching with lazy evaluation:

```rust
pub fn get_outgoing_edges(&self, from: &Address) -> Vec<(Address, Address, U256)> {
    if self.dirty_nodes.borrow().contains(from) {
        self.rebuild_cache_for_node(from);
    }
    // Return cached data...
}
```

This approach balances performance with memory usage, especially for large graphs.

### 4. Trust and Balance Management

The structure explicitly manages trust limits and token balances:

```rust
pub fn set_trust_limit(&mut self, from: Address, to: Address, trust_limit: U256) {
    // Implementation...
}

pub fn set_balance(&mut self, address: Address, token: Address, balance: U256) {
    // Implementation...
}
```

This allows for more accurate modeling of financial networks.

### 5. Flow Tracking

The structure keeps track of flows, enabling efficient updates during flow computation:

```rust
pub fn update_edge_capacity(&mut self, from: &Address, to: &Address, token: &Address, flow: U256) {
    // Implementation...
}
```

## Improvements Over the Old Implementation

1. **Flexibility**: The new structure is more adaptable to different types of network flow problems.
2. **Scalability**: Better handling of large graphs through efficient data structures and caching.
3. **Performance**: Lazy evaluation and caching strategies improve performance for large, sparse graphs.
4. **Clarity**: Clear separation between graph structure and algorithm logic.
5. **Maintainability**: More modular design makes it easier to update or extend functionality.

## Limitations and Considerations

1. **Memory Usage**: While more efficient than the old implementation, very large graphs may still require significant memory.
2. **Complexity**: The caching mechanisms add some complexity to the code.
3. **Concurrency**: The use of `RefCell` for caches may limit parallel processing capabilities.

## Conclusion

The new `FlowGraph` structure provides a solid foundation for implementing efficient and flexible network flow algorithms. Its design addresses many of the limitations of the old implementation while introducing new capabilities for handling complex and large-scale networks. This improved graph structure sets the stage for more advanced flow computation algorithms, which we'll explore in the subsequent sections.