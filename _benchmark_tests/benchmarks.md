---
layout: default
title: Benchmark Analysis
parent: Benchmark Tests
nav_order: 1
---

# Benchmark Analysis

This section provides a detailed analysis of benchmark tests comparing the performance of the original implementation against various algorithms in the new implementation across different types of transfers. These benchmarks quantify the improvements and identify areas for potential further optimization.

## Overview of Algorithms Tested

1. **Original Implementation**: The baseline algorithm used for comparison.

2. **Ford-Fulkerson (FF) Variants**:
   - FF_BFS: Ford-Fulkerson with Breadth-First Search
   - FF_BiBFS: Ford-Fulkerson with Bidirectional Breadth-First Search
   - FF_DFS: Ford-Fulkerson with Depth-First Search
   - FF_BiDFS: Ford-Fulkerson with Bidirectional Depth-First Search

3. **Capacity Scaling (CS) Variants**:
   - CS_BFS: Capacity Scaling with Breadth-First Search
   - CS_BiBFS: Capacity Scaling with Bidirectional Breadth-First Search
   - CS_DFS: Capacity Scaling with Depth-First Search
   - CS_BiDFS: Capacity Scaling with Bidirectional Depth-First Search


Each algorithm was tested across various the same transfer configurations to provide a comprehensive performance analysis.

The histogram below shows the distribution of execution times for all algorithm variations that were run, including the original implementation:

![Histogram of all algorithms]({{ site.baseurl }}/assets/figs/histogram_all.png)

## Performance Comparison Considerations

When comparing the models, it's crucial to consider that different algorithms may behave differently and produce varying results. In particular, an algorithm might appear to take longer than the original implementation when, in practice, the extra time allowed it to push more of the requested flow than the original version. We will examine this in more detail in subsequent sections.

## Overall Performance Distribution

![Overall performance boxplot]({{ site.baseurl }}/assets/figs/overall_boxplot.png)

The boxplot above provides an overview of the performance distribution for each algorithm. Key observations:

1. CS_BiBFS (Capacity Scaling with Bidirectional BFS) and FF_BiBFS (Ford-Fulkerson with Bidirectional BFS) appear to be the most promising strategies, showing competitive performance across various scenarios.
2. There's significant variation in performance across different algorithms, indicating that the choice of algorithm can have a substantial impact on execution time.
3. Some algorithms show wider interquartile ranges, suggesting they may be more sensitive to specific network configurations or flow requests.

## Performance Analysis for Matching Flow Scenarios

To provide a fair comparison, we focus on cases where the original script's flow matches the requested flow. This approach allows us to compare algorithms based on their ability to find optimal solutions within similar constraints.

![Histogram of matching flow scenarios]({{ site.baseurl }}/assets/figs/histogram_match_BiBFS.png)

Key findings from this analysis:

1. While the original implementation has some instances with lower execution times, CS_BiBFS and FF_BiBFS show distributions with much smaller tails, indicating more consistent performance.
2. The new implementations (CS_BiBFS and FF_BiBFS) appear to have a tighter distribution of execution times, suggesting more predictable performance across different scenarios.

## Performance Breakdown by Number of Transactions

To gain deeper insights, we analyze the performance distribution split by the number of transactions:

![Histogram for 1-10 transactions]({{ site.baseurl }}/assets/figs/histogram_1-10_transactions.png)
![Histogram for 11-100 transactions]({{ site.baseurl }}/assets/figs/histogram_11-100_transactions.png)
![Histogram for 100+ transactions]({{ site.baseurl }}/assets/figs/histogram_100+_transactions.png)

Observations:

1. For 1-10 transactions:
   - The original implementation can sometimes outperform the new algorithms.
   - However, the new algorithms show fewer long-tail events, indicating more consistent performance.

2. For 11-100 transactions:
   - CS_BiBFS and FF_BiBFS start to show clear advantages over the original implementation.
   - The performance gap widens, with the new algorithms demonstrating better consistency.

3. For 100+ transactions:
   - The new algorithms, especially CS_BiBFS, show significant performance improvements over the original implementation.
   - This suggests that the new implementations scale better for larger, more complex transfer scenarios.

## Analysis of Number of Transfers

We also examine the number of transfers required by each algorithm to find a solution:

![Transfers vs Transfers graph]({{ site.baseurl }}/assets/figs/trf_vs_trf.png)

Key findings:

1. In general, the new algorithms require more transactions than the original implementation.
2. This can be partially explained by the fact that in many cases, the new algorithms actually push more flow in the network than the original implementation, potentially finding more optimal solutions.

When we constrain the analysis to cases where there is a match between the flow pushed and the requested flow:

![Transfers vs Transfers graph for matching flows]({{ site.baseurl }}/assets/figs/trf_vs_trf_match.png)

Observations:

1. Capacity Scaling (CS) tends to require fewer transactions compared to the original implementation.
2. Ford-Fulkerson (FF) variants tend to require more transactions.
3. Some outliers are present and may require further investigation to understand the specific network conditions causing these extreme cases.

## Execution Time vs. Number of Transfers

![Execution Time vs Number of Transfers]({{ site.baseurl }}/assets/figs/line_time_vs_trf.png)

This graph provides insights into how execution time scales with the number of transfers:

1. For small numbers of transfers, all algorithms perform similarly.
2. As the number of transfers increases, the new algorithms (especially CS_BiBFS) show better scaling behavior compared to the original implementation.
3. The original implementation's execution time appears to increase more rapidly with the number of transfers, indicating potential scalability issues for larger, more complex networks.

## Conclusion and Future Work

The benchmarks demonstrate that the new implementation, particularly the Capacity Scaling with Bidirectional BFS (CS_BiBFS) algorithm, provides substantial performance improvements across various network types and sizes. Key conclusions:

1. CS_BiBFS and FF_BiBFS show excellent performance characteristics, especially for large-scale networks and complex transfer scenarios.
2. The new algorithms demonstrate more consistent performance, with fewer long-tail events in execution time.
3. While the new algorithms may sometimes require more transactions, they often push more flow, potentially finding more optimal solutions.
4. The performance improvements become more pronounced as the number of transactions increases, indicating better scalability for large-scale problems.

These results validate the design decisions made in the new implementation, such as the more efficient graph representation and the introduction of advanced algorithms. However, they also highlight areas for potential future work:

1. Further optimization of memory usage for very large dense networks.
2. Investigation of outlier cases to understand and potentially mitigate extreme performance scenarios.
3. Fine-tuning of algorithms to reduce the number of transactions while maintaining optimal flow in complex scenarios.
4. Exploration of hybrid approaches that combine the strengths of different algorithms based on network characteristics.

By continuing to refine these algorithms and their implementations, we can further improve the efficiency and effectiveness of network flow computations in decentralized financial systems.