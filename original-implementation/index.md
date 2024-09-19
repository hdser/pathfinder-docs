# Original Implementation

This section provides a detailed overview of the previous implementation of our network flow algorithms. This implementation, while functional, had certain limitations that led to the development of the new implementation.

## Contents

1. [Graph Structure](./graph-structure.md)
   - Node and Edge representation
   - EdgeDB and its functionalities

2. [Adjacencies](./adjacencies.md)
   - Adjacencies structure and its role
   - Lazy evaluation and caching strategies

3. [Flow Computation](./flow-computation.md)
   - Implementation of the Ford-Fulkerson algorithm
   - Augmenting path finding
   - Flow pruning and simplification

## Key Features

- Custom graph representation tailored for the specific needs of our financial network
- Lazy evaluation of adjacencies for potential performance benefits
- Implementation of flow pruning and transfer simplification

## Limitations

- Lack of support for different path search strategies
- Limited flexibility in choosing between different flow algorithms
- Potential performance issues with large, dense graphs

Understanding this implementation provides valuable context for the design decisions made in the new implementation and highlights the areas where improvements were sought.