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

### Advantages
- Finds the shortest path in terms of the number of edges.
- Guarantees optimality for unweighted graphs.
- Provides better theoretical time complexity when used in Ford-Fulkerson (Edmonds-Karp algorithm).

### Disadvantages
- May not be the most efficient for sparse graphs or when the sink is far from the source.

### Impact on Flow Algorithms
When used in Ford-Fulkerson (known as the Edmonds-Karp algorithm), it guarantees a polynomial time complexity of O(VE^2), where V is the number of vertices and E is the number of edges.

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

### Advantages
- Memory-efficient for very deep graphs.
- Can be faster than BFS for finding "any" path, especially in sparse graphs.

### Disadvantages
- Does not guarantee the shortest path.
- Can get stuck in deep branches of the graph.

### Impact on Flow Algorithms
When used in Ford-Fulkerson, it can lead to poor performance in certain graph structures, potentially resulting in a time complexity of O(E * max_flow).

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

### Advantages
- Can be significantly faster than standard BFS, especially when the source and sink are far apart.
- Still guarantees the shortest path.

### Disadvantages
- More complex to implement.
- Requires keeping track of two search frontiers.

### Impact on Flow Algorithms
Can potentially speed up each augmenting path finding step in Ford-Fulkerson or Capacity Scaling, especially in large, sparse networks.

## Comparison in the Context of Network Flow

1. **Path Length**: BFS and Bidirectional BFS find shortest augmenting paths, which is beneficial for the Edmonds-Karp algorithm. DFS may find longer paths, potentially leading to more iterations in Ford-Fulkerson.

2. **Performance**: 
   - BFS performs well in dense graphs and when short augmenting paths are common.
   - DFS can be faster in sparse graphs or when any path (not necessarily the shortest) is sufficient.
   - Bidirectional BFS often outperforms both in large networks, especially when source and sink are far apart.

3. **Theoretical Guarantees**: Using BFS in Ford-Fulkerson provides polynomial time complexity guarantees (Edmonds-Karp algorithm), while DFS does not.

4. **Memory Usage**: DFS generally uses less memory than BFS or Bidirectional BFS, which can be important for very large graphs.

In our implementation, we provide options for both BFS and Bidirectional BFS, allowing for performance comparisons in different network structures and flow scenarios. The choice between these algorithms can significantly impact the overall efficiency of the network flow computation, especially in large-scale financial networks with varying connectivity patterns.