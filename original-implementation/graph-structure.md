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
impl FlowGraph {
    pub fn add_edge(&mut self, from: Address, to: Address, token: Address, capacity: U256) {
        let key = EdgeKey { from, to, token };
        self.edges.insert(key.clone(), capacity);
        self.nodes.insert(from);
        self.nodes.insert(to);

        self.outgoing_edges.entry(from).or_default().push(key.clone());
        self.incoming_edges.entry(to).or_default().push(key);

        self.invalidate_cache(&from);
        self.invalidate_cache(&to);
    }

    pub fn remove_edge(&mut self, key: &EdgeKey) {
        if self.edges.remove(key).is_some() {
            if let Some(edges) = self.outgoing_edges.get_mut(&key.from) {
                edges.retain(|e| e != key);
            }
            if let Some(edges) = self.incoming_edges.get_mut(&key.to) {
                edges.retain(|e| e != key);
            }
        }

        self.invalidate_cache(&key.from);
        self.invalidate_cache(&key.to);
    }
}
```

These methods automatically update all relevant data structures and invalidate caches as necessary.

### 3. Caching with Lazy Evaluation

The `get_outgoing_edges` and `get_incoming_edges` methods use caching with lazy evaluation:

```rust
impl FlowGraph {
    pub fn get_outgoing_edges(&self, from: &Address) -> Vec<(Address, Address, U256)> {
        if self.dirty_nodes.borrow().contains(from) {
            self.rebuild_cache_for_node(from);
        }
    
        let mut edges = self.outgoing_edges_cache.borrow()
            .get(from)
            .cloned()
            .unwrap_or_default()
            .into_iter()
            .filter_map(|(to, token, capacity)| {
                let key = EdgeKey { from: *from, to, token };
                let adjustment = self.capacity_adjustments.get(&key).cloned().unwrap_or(U256::from(0));
                if capacity > adjustment {
                    Some((to, token, capacity - adjustment))
                } else {
                    None
                }
            })
            .collect::<Vec<_>>();

        edges.sort_by(|a, b| b.2.cmp(&a.2));
        edges
    }

    fn rebuild_cache_for_node(&self, node: &Address) {
        let mut outgoing_edges = Vec::new();
        let mut incoming_edges = Vec::new();
    
        if let Some(outgoing) = self.outgoing_edges.get(node) {
            for key in outgoing {
                let capacity = *self.edges.get(key).unwrap_or(&U256::from(0));
                outgoing_edges.push((key.to, key.token, capacity));
            }
        }
    
        if let Some(incoming) = self.incoming_edges.get(node) {
            for key in incoming {
                let capacity = *self.edges.get(key).unwrap_or(&U256::from(0));
                incoming_edges.push((key.from, key.token, capacity));
            }
        }
    
        self.outgoing_edges_cache.borrow_mut().insert(*node, outgoing_edges);
        self.incoming_edges_cache.borrow_mut().insert(*node, incoming_edges);
        self.dirty_nodes.borrow_mut().remove(node);
    }
}
```

This approach balances performance with memory usage, especially for large graphs.

### 4. Trust and Balance Management

The structure explicitly manages trust limits and token balances:

```rust
impl FlowGraph {
    pub fn set_trust_limit(&mut self, from: Address, to: Address, trust_limit: U256) {
        self.trust_limits.insert((from, to), trust_limit);
        self.invalidate_cache(&from);
        self.invalidate_cache(&to);
    }

    pub fn set_balance(&mut self, address: Address, token: Address, balance: U256) {
        self.balances.insert((address, token), balance);
        self.invalidate_cache(&address);
    }
}
```

This allows for more accurate modeling of financial networks.

### 5. Flow Tracking

The structure keeps track of flows, enabling efficient updates during flow computation:

```rust
impl FlowGraph {
    pub fn update_edge_capacity(&mut self, from: &Address, to: &Address, token: &Address, flow: U256) {
        let key = EdgeKey { from: *from, to: *to, token: *token };
        
        // Update edge capacity
        if let Some(capacity) = self.edges.get_mut(&key) {
            if *capacity > flow {
                *capacity -= flow;
            } else {
                self.edges.remove(&key);
                // Remove from outgoing and incoming edges
                if let Some(edges) = self.outgoing_edges.get_mut(from) {
                    edges.retain(|e| e != &key);
                }
                if let Some(edges) = self.incoming_edges.get_mut(to) {
                    edges.retain(|e| e != &key);
                }
            }
        }

        // Update balance
        if let Some(balance) = self.balances.get_mut(&(*from, *token)) {
            *balance = if *balance > flow { *balance - flow } else { U256::from(0) };
        }
        
        // Update trust limit, but not for return-to-owner
        if token != to {
            if let Some(trust_limit) = self.trust_limits.get_mut(&(*from, *to)) {
                *trust_limit = if *trust_limit > flow { *trust_limit - flow } else { U256::from(0) };
            }
        }
        
        // Update total flow
        let total_flow_key = (*from, *to);
        let total_flow = self.total_flows.entry(total_flow_key).or_insert(U256::from(0));
        *total_flow += flow;
        
        self.invalidate_cache(from);
        self.invalidate_cache(to);
    }
}
```

## Improvements Over the Old Implementation

1. **Flexibility**: The new structure is more adaptable to different types of network flow problems.

2. **Scalability**: Better handling of large graphs through efficient data structures and caching.

3. **Performance**: Lazy evaluation and caching strategies improve performance for large, sparse graphs.

4. **Clarity**: Clear separation between graph structure and algorithm logic.

5. **Maintainability**: More modular design makes it easier to update or extend functionality.

6. **Integrated Flow Tracking**: The new structure incorporates flow tracking, which was separate in the old implementation.

7. **Dynamic Updates**: Easier to perform and manage dynamic updates to the graph structure.

## Usage Example

Here's an example of how to use the new `FlowGraph` structure:

```rust
let mut graph = FlowGraph::new();

// Add edges
graph.add_edge(address1, address2, token1, U256::from(100));
graph.add_edge(address2, address3, token1, U256::from(50));

// Set trust limits and balances
graph.set_trust_limit(address1, address2, U256::from(200));
graph.set_balance(address1, token1, U256::from(150));

// Get outgoing edges
let edges = graph.get_outgoing_edges(&address1);
for (to, token, capacity) in edges {
    println!("Edge to {}: token {}, capacity {}", to, token, capacity);
}

// Update flow
graph.update_edge_capacity(&address1, &address2, &token1, U256::from(30));

// Check trust utilization
let utilization = graph.get_trust_utilization(&address1, &address2);
println!("Trust utilization: {}", utilization);
```

## Limitations and Considerations

1. **Memory Usage**: While more efficient than the old implementation, very large graphs may still require significant memory.

2. **Complexity**: The caching mechanisms add some complexity to the code.

3. **Concurrency**: The use of `RefCell` for caches may limit parallel processing capabilities.

## Conclusion

The new `FlowGraph` structure provides a solid foundation for implementing efficient and flexible network flow algorithms. Its design addresses many of the limitations of the old implementation while introducing new capabilities for handling complex and large-scale networks. 

The improvements in flexibility, scalability, and performance make it particularly well-suited for modeling and analyzing financial networks, where dynamic updates and efficient querying of large-scale data are crucial. The integrated flow tracking and trust limit management features allow for more accurate representation of financial relationships and constraints.

By providing a more robust and versatile graph structure, this new implementation sets the stage for more advanced and efficient flow computation algorithms, which we'll explore in subsequent sections.