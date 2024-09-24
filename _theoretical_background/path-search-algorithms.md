---
layout: default
title: Path Search Algorithms
parent: Theoretical Background
nav_order: 4
---

# Path Search Algorithms

Path search algorithms play a crucial role in network flow computations, particularly in finding augmenting paths for algorithms like Ford-Fulkerson and Capacity Scaling. The choice of path search algorithm can significantly impact the performance and behavior of the overall flow algorithm. In our project, we implement and compare several path search strategies.

## Breadth-First Search (BFS)

### Description
BFS explores the graph level by level, visiting all neighbors of a node before moving to the next level. It uses a queue data structure to keep track of nodes to visit.

### Algorithm Steps
1. Start at the source node and enqueue it.
2. While the queue is not empty:
   a. Dequeue a node.
   b. If this node is the sink, return the path.
   c. For each unvisited neighbor of the node:
      i. Mark it as visited.
      ii. Enqueue it.
      iii. Store its parent node for path reconstruction.
3. If the queue is empty and sink not reached, return no path found.

### Pseudocode
```python
def bfs(graph, source, sink):
    queue = [source]
    visited = {source: None}
    while queue:
        current = queue.pop(0)
        if current == sink:
            return reconstruct_path(visited, source, sink)
        for neighbor in graph.get_neighbors(current):
            if neighbor not in visited:
                visited[neighbor] = current
                queue.append(neighbor)
    return None  # No path found

def reconstruct_path(visited, source, sink):
    path = []
    current = sink
    while current != source:
        path.append(current)
        current = visited[current]
    path.append(source)
    return path[::-1]
```

### Time Complexity
- O(V + E), where V is the number of vertices and E is the number of edges.
- In the worst case, when the graph is complete, this becomes O(V^2).

### Space Complexity
- O(V) for the queue and visited set.

### Advantages
- Finds the shortest path in terms of the number of edges.
- Guarantees optimality for unweighted graphs.
- Provides better theoretical time complexity when used in Ford-Fulkerson (Edmonds-Karp algorithm).

### Disadvantages
- May not be the most efficient for sparse graphs or when the sink is far from the source.

## Depth-First Search (DFS)

### Description
DFS explores as far as possible along each branch before backtracking. It uses a stack (often implemented using recursion) to keep track of nodes to visit.

### Algorithm Steps
1. Start at the source node.
2. While the current node is not the sink:
   a. If the current node has unvisited neighbors:
      i. Choose an unvisited neighbor.
      ii. Mark it as visited.
      iii. Recursively apply DFS to this neighbor.
   b. If no unvisited neighbors, backtrack.
3. If sink is reached, return the path; otherwise, return no path found.

### Pseudocode
```python
def dfs(graph, source, sink, visited=None, path=None):
    if visited is None:
        visited = set()
    if path is None:
        path = []
    
    visited.add(source)
    path.append(source)
    
    if source == sink:
        return path
    
    for neighbor in graph.get_neighbors(source):
        if neighbor not in visited:
            result = dfs(graph, neighbor, sink, visited, path)
            if result is not None:
                return result
    
    path.pop()
    return None  # No path found
```

### Time Complexity
- O(V + E) in the worst case, where V is the number of vertices and E is the number of edges.
- However, the actual running time can vary significantly depending on the graph structure and the order of neighbor exploration.

### Space Complexity
- O(V) in the worst case for the recursion stack.

### Advantages
- Memory-efficient for very deep graphs.
- Can be faster than BFS for finding "any" path, especially in sparse graphs.

### Disadvantages
- Does not guarantee the shortest path.
- Can get stuck in deep branches of the graph.

## Bidirectional BFS

### Description
Bidirectional BFS runs two simultaneous BFS searches: one from the source and one from the sink, stopping when they meet in the middle.

### Algorithm Steps
1. Initialize two queues: one for the forward search from the source, one for the backward search from the sink.
2. Alternately perform one step of BFS from each direction:
   a. Dequeue a node.
   b. For each unvisited neighbor:
      i. Mark it as visited.
      ii. Enqueue it.
      iii. Store its parent node.
   c. If a node is encountered that has been visited by the other search, reconstruct and return the path.
3. If either queue becomes empty before a meeting point is found, return no path found.

### Pseudocode
```python
def bidirectional_bfs(graph, source, sink):
    forward_queue = [source]
    backward_queue = [sink]
    forward_visited = {source: None}
    backward_visited = {sink: None}
    
    while forward_queue and backward_queue:
        # Forward search
        current = forward_queue.pop(0)
        for neighbor in graph.get_neighbors(current):
            if neighbor not in forward_visited:
                forward_visited[neighbor] = current
                forward_queue.append(neighbor)
                if neighbor in backward_visited:
                    return reconstruct_bidirectional_path(forward_visited, backward_visited, source, sink, neighbor)
        
        # Backward search
        current = backward_queue.pop(0)
        for neighbor in graph.get_neighbors(current):
            if neighbor not in backward_visited:
                backward_visited[neighbor] = current
                backward_queue.append(neighbor)
                if neighbor in forward_visited:
                    return reconstruct_bidirectional_path(forward_visited, backward_visited, source, sink, neighbor)
    
    return None  # No path found

def reconstruct_bidirectional_path(forward_visited, backward_visited, source, sink, meeting_point):
    path = []
    current = meeting_point
    while current != source:
        path.append(current)
        current = forward_visited[current]
    path.append(source)
    path = path[::-1]
    
    current = backward_visited[meeting_point]
    while current != sink:
        path.append(current)
        current = backward_visited[current]
    path.append(sink)
    
    return path
```

### Time Complexity
- O(b^(d/2)), where b is the branching factor and d is the distance between source and sink.
- This is significantly better than the O(b^d) complexity of standard BFS, especially for large d.

### Space Complexity
- O(b^(d/2)) for storing the visited nodes from both directions.

### Advantages
- Can be significantly faster than standard BFS, especially when the source and sink are far apart.
- Still guarantees the shortest path.

### Disadvantages
- More complex to implement.
- Requires keeping track of two search frontiers.

## Impact on Flow Algorithm Performance

The choice of path search algorithm can significantly affect the performance of network flow algorithms:

1. **Ford-Fulkerson Algorithm**:
   - With BFS (Edmonds-Karp algorithm): Guarantees O(VE^2) time complexity.
   - With DFS: Can have poor performance, potentially O(E * max_flow) in the worst case.
   - With Bidirectional BFS: Can improve performance, especially in large, sparse networks.

2. **Capacity Scaling Algorithm**:
   - The impact of path search choice is less pronounced due to the Δ-residual graph restricting the search space.
   - BFS or Bidirectional BFS are generally preferred for their shortest path guarantee.

3. **Push-Relabel Algorithm**:
   - Less dependent on path search, as it works with local operations.
   - Path search is typically used in the "gap heuristic" for performance improvement.

### Empirical Observations

In our implementation, we've observed the following:

1. For dense networks with relatively uniform capacities, BFS tends to perform best due to its shortest path guarantee.

2. For sparse networks with varying capacities, Bidirectional BFS often outperforms standard BFS, especially when the source and sink are far apart.

3. DFS can sometimes find augmenting paths quickly in sparse networks but may lead to suboptimal flow distributions.

4. In the context of financial networks:
   - BFS and Bidirectional BFS tend to result in flow distributions that involve fewer intermediaries, which is often desirable.
   - The performance gap between BFS and Bidirectional BFS increases with network size, making Bidirectional BFS particularly valuable for large-scale financial networks.

## Considerations for Implementation

1. **Memory Usage**: BFS and Bidirectional BFS use more memory than DFS, which can be a concern for very large graphs.

2. **Path Length**: In financial networks, shorter paths (fewer intermediaries) are often preferred. BFS and Bidirectional BFS are advantageous in this regard.

3. **Graph Structure**: The optimal choice may depend on the specific structure of the financial network. It's beneficial to allow runtime selection of the path search algorithm.

4. **Parallelization**: Bidirectional BFS has potential for effective parallelization, which could be advantageous for large-scale computations.

In our implementation, we provide options for both BFS and Bidirectional BFS, allowing for performance comparisons in different network structures and flow scenarios. The choice between these algorithms can significantly impact the overall efficiency of the network flow computation, especially in large-scale financial networks with varying connectivity patterns.

By carefully selecting and optimizing the path search strategy, we can significantly enhance the performance of our network flow algorithms, leading to more efficient transaction routing in our decentralized financial system.