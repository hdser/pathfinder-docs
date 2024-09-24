---
layout: home
title: Home
nav_order: 1
---

# Network Flow Algorithms Documentation

Welcome to the documentation for our Network Flow Algorithms project. This documentation covers both the theoretical background and practical implementations of various network flow algorithms used in our system.

## Table of Contents

1. [Theoretical Background]({{ site.baseurl }}{% link _theoretical_background/index.md %})
   - [Network Flow Problem]({{ site.baseurl }}{% link _theoretical_background/network-flow-problem.md %})
   - [Ford-Fulkerson Algorithm]({{ site.baseurl }}{% link _theoretical_background/ford-fulkerson-algorithm.md %})
   - [Capacity Scaling Algorithm]({{ site.baseurl }}{% link _theoretical_background/capacity-scaling-algorithm.md %})
   - [Path Search Algorithms]({{ site.baseurl }}{% link _theoretical_background/path-search-algorithms.md %})

2. [Original Implementation]({{ site.baseurl }}{% link _original_implementation/index.md %})
   - [Graph Structure]({{ site.baseurl }}{% link _original_implementation/graph-structure.md %})
   - [Adjacencies]({{ site.baseurl }}{% link _original_implementation/adjacencies.md %})
   - [Flow Computation]({{ site.baseurl }}{% link _original_implementation/flow-computation.md %})

3. [New Implementation]({{ site.baseurl }}{% link _new_implementation/index.md %})
   - [Graph Structure]({{ site.baseurl }}{% link _new_implementation/graph-structure.md %})
   - [Path Search]({{ site.baseurl }}{% link _new_implementation/path-search.md %})
   - [Ford-Fulkerson Implementation]({{ site.baseurl }}{% link _new_implementation/ford-fulkerson.md %})
   - [Capacity Scaling Implementation]({{ site.baseurl }}{% link _new_implementation/capacity-scaling.md %})
   - [Flow Recorder]({{ site.baseurl }}{% link _new_implementation/flow-recorder.md %})

4. [Comparison]({{ site.baseurl }}{% link _benchmark_tests/index.md %})
   - [Benchmarks]({{ site.baseurl }}{% link _benchmark_tests/benchmarks.md %})

This documentation aims to provide a comprehensive understanding of the network flow algorithms used in the pathfinder2, including both theoretical foundations and practical implementations. It covers both the original and new implementations, allowing for comparison and analysis of the different approaches.