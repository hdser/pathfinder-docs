---
layout: default
title: Flow Computation
parent: Original Implementation
nav_order: 3
---

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

## Flow Diagram

Here's a high-level flow diagram of the algorithm:

```
┌─────────────────┐
│ Initialize Flow │
└────────┬────────┘
         │
         v
┌─────────────────┐
│  Find Aug Path  │◄─────────────────┐
└────────┬────────┘                  │
         │                           │
         v                           │
┌─────────────────┐         No       │
│ Path Found?     ├─────────────────►│
└────────┬────────┘                  │
         │ Yes                       │
         v                           │
┌─────────────────┐                  │
│  Augment Flow   │                  │
└────────┬────────┘                  │
         │                           │
         v                           │
┌─────────────────┐         No       │
│ Max Flow/Req?   ├─────────────────►│
└────────┬────────┘                  │
         │ Yes                       │
         v                           │
┌─────────────────┐                  │
│   Prune Flow    │                  │
└────────┬────────┘                  │
         │                           │
         v                           │
┌─────────────────┐                  │
│ Reduce Transfers│                  │
└────────┬────────┘                  │
         │                           │
         v                           │
┌─────────────────┐
│ Extract & Sort  │
│   Transfers     │
└────────┬────────┘
         │
         v
┌─────────────────┐
│  Return Result  │
└─────────────────┘
```

## Key Components

### 1. Augmenting Path Finding

The algorithm used a modified breadth-first search to find augmenting paths:

```rust
fn augmenting_path(
    source: &Address,
    sink: &Address,
    adjacencies: &mut Adjacencies,
    max_distance: Option<u64>,
) -> (U256, Vec<Node>) {
    let mut parent = HashMap::new();
    if *source == *sink {
        return (U256::default(), vec![]);
    }
    let mut queue = VecDeque::<(Node, (u64, U256))>::new();
    queue.push_back((Node::Node(*source), (0, U256::default() - U256::from(1))));
    while let Some((node, (depth, flow))) = queue.pop_front() {
        if let Some(max) = max_distance {
            if depth >= max * 3 {
                continue;
            }
        }
        for (target, capacity) in adjacencies.outgoing_edges_sorted_by_capacity(&node) {
            if !parent.contains_key(&target) && capacity > U256::default() {
                parent.insert(target.clone(), node.clone());
                let new_flow = min(flow, capacity);
                if target == Node::Node(*sink) {
                    return (
                        new_flow,
                        trace(parent, &Node::Node(*source), &Node::Node(*sink)),
                    );
                }
                queue.push_back((target, (depth + 1, new_flow)));
            }
        }
    }
    (U256::default(), vec![])
}
```

This function returned the flow value and the path found. It respected the `max_distance` parameter to limit the search depth.

### 2. Flow Augmentation

After finding an augmenting path, the algorithm updated the flow:

```rust
for window in parents.windows(2) {
    if let [node, prev] = window {
        adjacencies.adjust_capacity(prev, node, -new_flow);
        adjacencies.adjust_capacity(node, prev, new_flow);
        if adjacencies.is_adjacent(node, prev) {
            *used_edges
                .entry(node.clone())
                .or_default()
                .entry(prev.clone())
                .or_default() -= new_flow;
        } else {
            *used_edges
                .entry(prev.clone())
                .or_default()
                .entry(node.clone())
                .or_default() += new_flow;
        }
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

fn prune_flow(
    source: &Address,
    sink: &Address,
    mut flow_to_prune: U256,
    used_edges: &mut HashMap<Node, HashMap<Node, U256>>,
) -> U256 {
    let edges_by_path_length = compute_edges_by_path_length(source, sink, used_edges);

    for edges_here in edges_by_path_length.values() {
        while flow_to_prune > U256::from(0) && !edges_here.is_empty() {
            if let Some((s, t)) = smallest_edge_in_set(used_edges, edges_here) {
                if used_edges[&s][&t] > flow_to_prune {
                    break;
                };
                flow_to_prune = prune_edge(used_edges, (&s, &t), flow_to_prune);
            } else {
                break;
            }
        }
    }
    // If there is still flow to prune, take the first element in edgesByPathLength
    // and partially prune its path.
    if flow_to_prune > U256::from(0) {
        for edges_here in edges_by_path_length.values() {
            for (a, b) in edges_here {
                if !used_edges.contains_key(a) || !used_edges[a].contains_key(b) {
                    continue;
                }
                flow_to_prune = prune_edge(used_edges, (a, b), flow_to_prune);
                if flow_to_prune == U256::from(0) {
                    return U256::from(0);
                }
            }
            if flow_to_prune == U256::from(0) {
                return U256::from(0);
            }
        }
    }
    flow_to_prune
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

fn reduce_transfers(
    max_transfers: u64,
    used_edges: &mut HashMap<Node, HashMap<Node, U256>>,
) -> U256 {
    let mut reduced_flow = U256::from(0);
    while used_edges.len() > max_transfers as usize {
        let all_edges = used_edges
            .iter()
            .flat_map(|(f, e)| e.iter().map(|(t, c)| ((f.clone(), t.clone()), c)));
        if all_edges.clone().count() <= max_transfers as usize {
            return reduced_flow;
        }
        let ((f, t), c) = all_edges
            .min_by_key(|(addr, c)| (*c, addr.clone()))
            .unwrap();
        reduced_flow += *c;
        prune_edge(used_edges, (&f, &t), *c);
    }
    reduced_flow
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

fn extract_transfers(
    source: &Address,
    sink: &Address,
    amount: &U256,
    mut used_edges: HashMap<Node, HashMap<Node, U256>>,
) -> Vec<Edge> {
    let mut transfers: Vec<Edge> = Vec::new();
    let mut account_balances: BTreeMap<Address, U256> = BTreeMap::new();
    account_balances.insert(*source, *amount);

    while !account_balances.is_empty()
        && (account_balances.len() > 1 || *account_balances.iter().next().unwrap().0 != *sink)
    {
        let edge = next_full_capacity_edge(&used_edges, &account_balances);
        assert!(account_balances[&edge.from] >= edge.capacity);
        account_balances
            .entry(edge.from)
            .and_modify(|balance| *balance -= edge.capacity);
        *account_balances.entry(edge.to).or_default() += edge.capacity;
        account_balances.retain(|_account, balance| balance > &mut U256::from(0));
        assert!(used_edges.contains_key(&Node::BalanceNode(edge.from, edge.token)));
        used_edges
            .entry(Node::BalanceNode(edge.from, edge.token))
            .and_modify(|outgoing| {
                assert!(outgoing.contains_key(&Node::TrustNode(edge.to, edge.token)));
                outgoing.remove(&Node::TrustNode(edge.to, edge.token));
            });
        transfers.push(edge);
    }

    transfers
}
```

The `extract_transfers` function converted the internal flow representation into a series of `Edge` structs representing individual transfers.

### 6. Transfer Simplification

The extracted transfers were then simplified to reduce the number of hops:

```rust
let simplified_transfers = simplify_transfers(transfers);

fn simplify_transfers(mut transfers: Vec<Edge>) -> Vec<Edge> {
    while let Some((i, j)) = find_pair_to_simplify(&transfers) {
        transfers[i].to = transfers[j].to;
        transfers.remove(j);
    }
    transfers
}

fn find_pair_to_simplify(transfers: &Vec<Edge>) -> Option<(usize, usize)> {
    let l = transfers.len();
    (0..l)
        .flat_map(move |x| (0..l).map(move |y| (x, y)))
        .find(|(i, j)| {
            let a = transfers[*i];
            let b = transfers[*j];
            *i != *j && a.to == b.from && a.token == b.token && a.capacity == b.capacity
        })
}
```

This step combined consecutive transfers where possible, reducing the overall number of transactions needed to achieve the computed flow.

### 7. Transfer Sorting

Finally, the transfers were sorted to ensure they could be executed in a valid order:

```rust
let sorted_transfers = sort_transfers(simplified_transfers);

fn sort_transfers(transfers: Vec<Edge>) -> Vec<Edge> {
    let mut receives_to_wait_for: HashMap<Address, u64> = HashMap::new();
    for e in &transfers {
        *receives_to_wait_for.entry(e.to).or_default() += 1;
        receives_to_wait_for.entry(e.from).or_default();
    }
    let mut result = Vec::new();
    let mut queue = transfers.into_iter().collect::<VecDeque<Edge>>();
    while let Some(e) = queue.pop_front() {
        if *receives_to_wait_for.get(&e.from).unwrap() == 0 {
            *receives_to_wait_for.get_mut(&e.to).unwrap() -= 1;
            result.push(e)
        } else {
            queue.push_back(e);
        }
    }
    result
}
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