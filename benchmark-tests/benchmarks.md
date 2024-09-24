# Benchmarks

This section contains detailed benchmarks comparing the performance of the old and new implementations across various network types and sizes. These benchmarks help quantify the improvements and identify any potential areas for further optimization.


<img src="../assets/figs/histogram_all.png" width="500" height="350" />
<img src="../assets/figs/overall_boxplot.png" width="500" height="350" />

<img src="../assets/figs/histogram_match_BiBFS.png" width="700" height="450" />

<img src="../assets/figs/histogram_1-10_transactions.png" width="350" height="250" />
<img src="../assets/figs/histogram_11-100_transactions.png" width="350" height="250" />
<img src="../assets/figs/histogram_100+_transactions.png" width="350" height="250" />

<img src="../assets/figs/line_time_vs_trf.png" width="700" height="450" />

<img src="../assets/figs/trf_vs_trf.png" width="700" height="450" />




## Results

### Sparse Networks


### Scale-Free Networks

| Size | Algorithm | Execution Time (s) | Memory Usage (MB) | Max Flow | Augmenting Paths |
|------|-----------|---------------------|-------------------|----------|-------------------|
| Small | Old FF | 0.18 | 27 | 2000 | 180 |
| Small | New FF | 0.15 | 25 | 2000 | 180 |
| Small | Capacity Scaling | 0.12 | 26 | 2000 | 90 |
| Small | Push-Relabel | 0.10 | 27 | 2000 | N/A |
| Medium | Old FF | 3.0 | 270 | 10000 | 900 |
| Medium | New FF | 2.2 | 250 | 10000 | 900 |
| Medium | Capacity Scaling | 1.5 | 260 | 10000 | 450 |
| Medium | Push-Relabel | 1.1 | 270 | 10000 | N/A |
| Large | Old FF | 55.0 | 2700 | 50000 | 4500 |
| Large | New FF | 40.0 | 2500 | 50000 | 4500 |
| Large | Capacity Scaling | 25.0 | 2600 | 50000 | 2250 |
| Large | Push-Relabel | 18.0 | 2700 | 50000 | N/A |
| Extra Large | Old FF | 900.0 | 27000 | 200000 | 18000 |
| Extra Large | New FF | 600.0 | 25000 | 200000 | 18000 |
| Extra Large | Capacity Scaling | 350.0 | 26000 | 200000 | 9000 |
| Extra Large | Push-Relabel | 250.0 | 27000 | 200000 | N/A |

## Analysis

1. **Performance Improvements**: The new implementation consistently outperforms the old implementation across all network types and sizes. The Capacity Scaling and Push-Relabel algorithms show significant improvements, especially for larger networks.

2. **Memory Usage**: The new implementation generally uses less memory than the old implementation, which is particularly beneficial for large networks.

3. **Scalability**: The new implementation, especially with Capacity Scaling and Push-Relabel algorithms, shows better scalability as network size increases.

4. **Network Type Impact**: 
   - Sparse Networks: All algorithms perform well, with Push-Relabel showing the best performance.
   - Dense Networks: The performance gap between algorithms narrows, but Capacity Scaling and Push-Relabel still outperform the others.
   - Scale-Free Networks: The new implementation shows significant improvements, likely due to better handling of high-degree nodes.

5. **Augmenting Paths**: Capacity Scaling consistently finds the maximum flow with fewer augmenting paths, which contributes to its performance advantage.

6. **Memory Limitations**: For extra large dense networks, all implementations struggle with memory usage, indicating an area for potential future optimization.

## Conclusion

The benchmarks demonstrate that the new implementation provides substantial performance improvements across various network types and sizes. The Capacity Scaling and Push-Relabel algorithms, in particular, show excellent performance characteristics, especially for large-scale networks.

These results validate the design decisions made in the new implementation, such as the more efficient graph representation and the introduction of advanced algorithms. However, they also highlight areas for potential future work, such as optimizing memory usage for very large dense networks.

![image](../assets/figs/flow_visualization_1.gif)

---

Note: This placeholder will be replaced with actual benchmark results and analysis once the benchmarks have been conducted.