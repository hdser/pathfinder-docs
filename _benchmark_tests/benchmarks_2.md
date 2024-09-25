---
layout: default
title: Benchmark Analysis 2
parent: Benchmark Tests
nav_order: 2
---

# Benchmark Analysis 2

In this study case we will look at the flow transfer between address `0x9BA1Bcd88E99d6E1E03252A70A63FEa83Bf1208c` and `0x42cEDde51198D1773590311E2A340DC06B24cB37`. 

In the first case we look a small transfer value 1000000000000000000 (i.e. 1 CRC). Both orignial and new implementations lead to the same flow graph (displayed below).

![Transfers vs Transfers graph for matching flows]({{ site.baseurl }}/assets/figs/flow_graph_1.png)

When the value is increased to 90000000000000000000 (i.e. 90 CRC) we see that the two implementations diverge. The original implementation pushes the full flow trhough a an intermediary trust, however in the new version there is still 87389957741691338102 (e.i. 87.3899 CRC) that are transfered directly and only the remaining is pushed via the intermediary node

![Transfers vs Transfers graph for matching flows]({{ site.baseurl }}/assets/figs/flow_graph_2_original.png)
![Transfers vs Transfers graph for matching flows]({{ site.baseurl }}/assets/figs/flow_graph_2_CSBiBFS.png)

