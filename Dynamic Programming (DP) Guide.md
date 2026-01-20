# Dynamic Programming (DP) Guide
---
## **1. What is Dynamic Programming?**
- **Definition**: A method to solve complex problems by breaking them into overlapping subproblems, solving each subproblem once, and storing results (memoization/tabulation) to avoid redundant computations.
- **Key Properties**:
  - **Optimal Substructure**: Optimal solutions build on smaller subproblems.
  - **Overlapping Subproblems**: Subproblems recur multiple times.

**Example**: Fibonacci Sequence  
- Without DP: Exponential time.  
- With DP: Linear time via stored results.

---

## **2. Types of Dynamic Programming**
### **Top-Down (Memoization)**
- Recursive approach with cached results.
- Example: Fibonacci with memoization.
  ```python
  def fib(n, memo={}):
      if n in memo: return memo[n]
      if n <= 2: return 1
      memo[n] = fib(n-1, memo) + fib(n-2, memo)
      return memo[n]
    ```
Bottom-Up (Tabulation)
Iterative approach solving subproblems first.

Example: Fibonacci with tabulation.
def fib(n):
    if n <= 2: return 1
    dp = [0]*(n+1)
    dp[1], dp[2] = 1, 1
    for i in range(3, n+1):
        dp[i] = dp[i-1] + dp[i-2]
    return dp[n]

Common Variations
1D DP: Coin Change, Fibonacci.
2D DP: Longest Common Subsequence (LCS), 0/1 Knapsack.
State Machine DP: Stock trading with cooldown.

3. Applications of DP
Computer Science: Shortest path algorithms (Floyd-Warshall), string alignment (edit distance).
Operations Research: Resource allocation.

Economics: Optimal saving strategies.
Bioinformatics: DNA sequence alignment.

Popular Problems:
0/1 Knapsack
Longest Increasing Subsequence (LIS)
Coin Change (Unbounded/Bounded)
Matrix Chain Multiplication
Edit Distance

4. How to Use DP?
Identify Subproblems: Look for overlapping subproblems and optimal substructure.
Define State: Use variables like dp[i][j] to represent the problem state.
Formulate Recurrence: Derive a relation between states (e.g., dp[i] = dp[i-1] + dp[i-2]).
Set Base Cases: Initialize trivial cases (e.g., dp[0] = 0).
Implement: Choose between top-down (recursive + memo) or bottom-up (iterative).

5. Key Insights
Time-Space Trade-off: Memoization (recursive stack) vs. tabulation (array storage).
Space Optimization: Reduce dimensions (e.g., use rolling arrays).
Greedy vs. DP: Greedy works if no overlapping subproblems.
Practice Patterns: Look for "max/min," "count ways," or "yes/no decisions."

6. Code Examples
```python
0/1 Knapsack (Bottom-Up)
def knapsack(values, weights, capacity):
    n = len(values)
    dp = [[0]*(capacity+1) for _ in range(n+1)]
    for i in range(1, n+1):
        for w in range(1, capacity+1):
            if weights[i-1] > w:
                dp[i][w] = dp[i-1][w]
            else:
                dp[i][w] = max(dp[i-1][w], values[i-1] + dp[i-1][w - weights[i-1]])
    return dp[n][capacity]

Longest Increasing Subsequence (LIS)
def length_of_lis(nums):
    dp = [1] * len(nums)
    for i in range(len(nums)):
        for j in range(i):
            if nums[j] < nums[i]:
                dp[i] = max(dp[i], dp[j] + 1)
    return max(dp)
```
7. Common Mistakes
Incorrect recurrence relations.
Missing base cases (e.g., forgetting to initialize dp[0]).
Off-by-one errors in tabulation indices.
Pro Tip: Practice identifying patterns like "subset selection," "sequence alignment," and "state transitions" to ace DP interviews!

---------------------------------------------------------------------------------------



# Dynamic Programming Practice Problems (48 Applications)

| Problem Name (Abbreviation) | Practice Problems | Key Insights | Recurrence Relation | Time Complexity |
|-----------------------------|-------------------|--------------|----------------------|-----------------|
| **Optimal Allotment (ALLOT)** | [Budget Allocation](https://leetcode.com/problems/maximum-profit-in-job-scheduling/) | Distribute resources to maximize profit | `dp[i][j] = max(dp[i-1][j-k] + profit[i][k])` | O(n·m²) |
| **All-Pairs Shortest Path (APSP)** | [Floyd-Warshall Implementation](https://www.geeksforgeeks.org/floyd-warshall-algorithm-dp-16/), [Cheapest Flights](https://leetcode.com/problems/cheapest-flights-within-k-stops/) | All nodes shortest paths | `dp[i][j] = min(dp[i][j], dp[i][k] + dp[k][j])` | O(V³) |
| **Optimal Alphabetic Radix-Code Tree (ARC)** | [Huffman Coding](https://www.geeksforgeeks.org/huffman-coding-greedy-algo-3/), [Optimal Merge Pattern](https://practice.geeksforgeeks.org/problems/minimum-cost-of-ropes-1587115620/1) | Minimal weighted path length | `dp[i][j] = min(dp[i][k] + dp[k+1][j] + sum_freq(i,j))` | O(n³) |
| **Assembly Line Balancing (ASMBAL)** | [Car Assembly](https://www.geeksforgeeks.org/assembly-line-scheduling-dp-34/), [Factory Optimization](https://leetcode.com/discuss/interview-question/1023609/assembly-line-balancing) | Minimize assembly time with switch costs | `dp[line][station] = time + min(prev_line)` | O(n·k) |
| **Optimal Assignment (ASSIGN)** | [Job Assignment](https://practice.geeksforgeeks.org/problems/job-assignment-problem/0), [Worker Tasks](https://leetcode.com/problems/maximum-compatibility-score-sum/) | Minimize assignment cost | `dp[mask] = min(cost[i][j] + dp[mask ^ (1<<j)])` | O(n·2ⁿ) |
| **Optimal Binary Search Tree (BST)** | [OBST Construction](https://www.geeksforgeeks.org/optimal-binary-search-tree-dp-24/), [Search Cost](https://leetcode.com/discuss/interview-question/operational/optimal-binary-search-tree) | Minimize search cost | `dp[i][j] = min(dp[i][k-1] + dp[k+1][j] + sum_freq(i,j))` | O(n³) |
| **Optimal Covering (COV)** | [Jump Game II](https://leetcode.com/problems/jump-game-ii/), [Video Stitching](https://leetcode.com/problems/video-stitching/) | Cover range with minimal segments | `dp[i] = min(dp[j] + 1)` for covering j | O(n²) |
| **Deadline Scheduling (DEADLINE)** | [Maximum Profit Jobs](https://leetcode.com/problems/maximum-profit-in-job-scheduling/), [Task Scheduler](https://leetcode.com/problems/task-scheduler/) | Schedule before deadlines | `dp[t] = max(dp[t], dp[t-time] + profit)` | O(n·d) |
| **Discounted Profits (DPP)** | [Best Time to Buy Stocks](https://leetcode.com/problems/best-time-to-buy-and-sell-stock-with-cooldown/), [Investment Planning](https://leetcode.com/discuss/interview-question/operational/discounted-profits) | Time-discounted profits | `dp[t] = max(do activity, skip)` | O(n) |
| **Edit Distance (EDP)** | [Levenshtein Distance](https://leetcode.com/problems/edit-distance/), [One Edit Distance](https://leetcode.com/problems/one-edit-distance/) | String transformation | `dp[i][j] = dp[i-1][j-1] if match else 1 + min(ops)` | O(m·n) |
| **Fibonacci (FIB)** | [Climbing Stairs](https://leetcode.com/problems/climbing-stairs/), [N-th Tribonacci](https://leetcode.com/problems/n-th-tribonacci-number/) | Classic memoization | `dp[n] = dp[n-1] + dp[n-2]` | O(n) |
| **Flowshop (FLOWSHOP)** | [Job Sequencing](https://www.geeksforgeeks.org/flow-shop-scheduling/), [Permutation Scheduling](https://leetcode.com/discuss/interview-question/operational/flowshop) | Minimize makespan | Permutation-based DP | O(n!·n) |
| **Tower of Hanoi (HANOI)** | [Classic Puzzle](https://www.geeksforgeeks.org/c-program-for-tower-of-hanoi/), [Minimum Moves](https://leetcode.com/discuss/interview-question/operational/tower-of-hanoi) | Recursive disk moves | `dp[n] = 2·dp[n-1] + 1` | O(n) |
| **Integer Linear Programming (ILP)** | [Knapsack as ILP](https://www.geeksforgeeks.org/0-1-knapsack-problem-dp-10/), [Production Planning](https://leetcode.com/discuss/interview-question/operational/integer-programming) | Solve ILP via DP | Problem-specific | Exponential |
| **Integer Knapsack (ILPKNAP)** | [Coin Change](https://leetcode.com/problems/coin-change/), [Unbounded Knapsack](https://www.geeksforgeeks.org/unbounded-knapsack-repetition-items-allowed/) | Unlimited items | `dp[w] = max(dp[w], dp[w-wt[i]] + val[i])` | O(n·W) |
| **Interval Scheduling (INTVL)** | [Non-overlapping Intervals](https://leetcode.com/problems/non-overlapping-intervals/), [Meeting Rooms III](https://leetcode.com/problems/meeting-rooms-iii/) | Maximize non-overlapping | `dp[i] = max(dp[i-1], dp[prev] + profit[i])` | O(n log n) |
| **Inventory (INVENT)** | [Stock Management](https://leetcode.com/discuss/interview-question/operational/inventory-management), [Order Planning](https://www.geeksforgeeks.org/solving-inventory-management-using-dynamic-programming/) | Optimal stock levels | `dp[t][i] = min(order_cost + dp[t+1][j])` | O(T·n²) |
| **Optimal Investment (INVEST)** | [Portfolio Optimization](https://leetcode.com/discuss/interview-question/operational/investment), [Budget Allocation](https://www.geeksforgeeks.org/investment-management-using-dp/) | Maximize returns | `dp[i][j] = max(dp[i-1][j-k] + return[i][k])` | O(n·m²) |
| **Las Vegas Investment (INVESTWLV)** | [Gambling Strategy](https://leetcode.com/discuss/interview-question/operational/las-vegas), [Probability DP](https://www.geeksforgeeks.org/probability-of-reaching-a-point-with-2-or-3-steps-at-a-time/) | Betting optimization | Probability-based DP | O(n·B) |
| **0/1 Knapsack (KS01)** | [Classic Knapsack](https://leetcode.com/problems/partition-equal-subset-sum/), [Target Sum](https://leetcode.com/problems/target-sum/) | Include/exclude items | `dp[i][w] = max(dp[i-1][w], dp[i-1][w-wt[i]] + val[i])` | O(n·W) |
| **COV as KSINT (KSCOV)** | [Cover with Costs](https://leetcode.com/discuss/interview-question/operational/covering-knapsack), [Modified Knapsack](https://www.geeksforgeeks.org/knapsack-with-large-weights/) | Knapsack formulation | Similar to KSINT | O(n·W) |
| **Integer Knapsack (KSINT)** | [Coin Change II](https://leetcode.com/problems/coin-change-ii/), [Rod Cutting](https://www.geeksforgeeks.org/cutting-a-rod-dp-13/) | Unlimited items | `dp[w] = max(dp[w], dp[w-wt[i]] + val[i])` | O(n·W) |
| **Longest Common Subseq. (LCS)** | [LCS Classic](https://leetcode.com/problems/longest-common-subsequence/), [Delete Operation](https://leetcode.com/problems/delete-operation-for-two-strings/) | Subsequence matching | `dp[i][j] = 1 + dp[i-1][j-1] if match else max(up,left)` | O(m·n) |
| **Optimal Linear Search (LINSRC)** | [Binary Search Cost](https://leetcode.com/discuss/interview-question/operational/optimal-search), [Weighted Search](https://www.geeksforgeeks.org/optimal-binary-search-tree-dp-24/) | Minimize search cost | `dp[i][j] = min(dp[i][k-1] + dp[k+1][j]) + sum_weights` | O(n³) |
| **Lot Size (LOT)** | [Production Batches](https://leetcode.com/discuss/interview-question/operational/lot-sizing), [Inventory Orders](https://www.geeksforgeeks.org/solving-production-scheduling-using-dp/) | Batch optimization | `dp[t] = min(setup_cost + production_cost + dp[t+1])` | O(T²) |
| **Longest Simple Path (LSP)** | [DAG Longest Path](https://www.geeksforgeeks.org/find-longest-path-directed-acyclic-graph/), [Critical Path](https://leetcode.com/discuss/interview-question/operational/longest-path) | Path maximization | `dp[v] = max(dp[u] + weight(u,v))` | O(V+E) |
| **Matrix Chain Mult. (MCM)** | [Chain Multiplication](https://leetcode.com/problems/minimum-score-triangulation-of-polygon/), [Burst Balloons](https://leetcode.com/problems/burst-balloons/) | Optimal parentheses | `dp[i][j] = min(dp[i][k] + dp[k+1][j] + cost)` | O(n³) |
| **Minimum Maximum (MINMAX)** | [Split Array Largest Sum](https://leetcode.com/problems/split-array-largest-sum/), [Painter's Partition](https://www.geeksforgeeks.org/painters-partition-problem/) | Partition optimization | `dp[k][i] = min(max(dp[k-1][j], sum[j+1..i]))` | O(k·n²) |
| **Min Weight Spanning Tree (MWST)** | [Prim's Algorithm](https://www.geeksforgeeks.org/prims-minimum-spanning-tree-mst-greedy-algo-5/), [Kruskal's](https://leetcode.com/problems/min-cost-to-connect-all-points/) | Tree with minimal weight | Specialized DP | O(E log V) |
| **Game of NIM (NIM)** | [Stone Game](https://leetcode.com/problems/stone-game/), [Nim Game](https://leetcode.com/problems/nim-game/) | Winning states | `dp[state] = OR(not dp[state - move])` | O(2ⁿ) |
| **Optimal Distribution (ODP)** | [Candy Distribution](https://leetcode.com/problems/candy/), [Resource Allocation](https://www.geeksforgeeks.org/resource-allocation-graph-rag-in-operating-system/) | Fair distribution | `dp[i][j] = min(dp[i-1][j-k] + cost[k])` | O(n·m²) |
| **Optimal Permutation (PERM)** | [Count Permutations](https://leetcode.com/problems/permutation-sequence/), [TSP Variant](https://www.geeksforgeeks.org/travelling-salesman-problem-set-1/) | Permutation optimization | `dp[mask] = min(cost[i][j] + dp[mask ^ (1<<j)])` | O(n·2ⁿ) |
| **Jug-Pouring (POUR)** | [Water Jug](https://leetcode.com/problems/water-and-jug-problem/), [Measure Water](https://www.geeksforgeeks.org/two-water-jug-puzzle/) | State transitions | BFS/DP with states | O(a·b) |
| **Optimal Production (PROD)** | [Manufacturing Planning](https://leetcode.com/discuss/interview-question/operational/production-planning), [Multi-period](https://www.geeksforgeeks.org/production-scheduling-problem/) | Production scheduling | `dp[t][i] = min(production_cost + dp[t+1][j])` | O(T·n²) |
| **Reject Allowances (PRODRAP)** | [Quality Control](https://leetcode.com/discuss/interview-question/operational/reject-allowances), [Defective Items](https://www.geeksforgeeks.org/reject-allowances-problem/) | Profit with rejects | `dp[i][j] = max(dp[i-1][j-k] + profit[k])` | O(n·m²) |
| **Reliability Design (RDP)** | [System Reliability](https://www.geeksforgeeks.org/reliability-design-problem-dynamic-programming/), [Component Redundancy](https://leetcode.com/discuss/interview-question/operational/reliability) | Maximize reliability | `dp[c][r] = min(cost + dp[c-1][r-reliability])` | O(C·R) |
| **Replacement (REPLACE)** | [Machine Replacement](https://leetcode.com/discuss/interview-question/operational/replacement), [Equipment Upgrade](https://www.geeksforgeeks.org/replacement-problem/) | Cost minimization | `dp[t] = min(keep_cost + dp[t+1], replace_cost + dp[1])` | O(T) |
| **Stagecoach (SCP)** | [Shortest Path Stages](https://www.geeksforgeeks.org/stagecoach-problem-dynamic-programming/), [Multi-stage Graph](https://leetcode.com/discuss/interview-question/operational/stagecoach) | Staged transitions | `dp[stage][node] = min(prev_stage + transition)` | O(S·N²) |
| **Disk Scheduling (SEEK)** | [Head Movement](https://www.geeksforgeeks.org/disk-scheduling-algorithms/), [SCAN Algorithm](https://leetcode.com/discuss/interview-question/operational/disk-scheduling) | Minimize seeks | `dp[position][requests_left]` | O(n·m) |
| **Segmented Curve Fitting (SEGLINE)** | [Piecewise Linear](https://leetcode.com/discuss/interview-question/operational/curve-fitting), [Approximation](https://www.geeksforgeeks.org/segmented-least-squares/) | Minimize error | `dp[i][j] = min(dp[k][j-1] + error(k+1,i))` | O(n²·k) |
| **Program Segmentation (SEGPAGE)** | [Memory Paging](https://www.geeksforgeeks.org/program-segmentation/), [Optimal Segments](https://leetcode.com/discuss/interview-question/operational/segmentation) | Page fault minimization | `dp[i] = min(dp[j] + penalty(j+1,i))` | O(n²) |
| **Optimal Selection (SELECT)** | [Subset Selection](https://leetcode.com/problems/subsets-ii/), [Constraint Satisfaction](https://www.geeksforgeeks.org/optimal-substructure-property-in-dynamic-programming-dp-2/) | Constrained choices | Problem-specific | Varies |
| **Shortest Path Acyclic (SPA)** | [DAG Shortest Path](https://www.geeksforgeeks.org/shortest-path-for-directed-acyclic-graphs/), [Topological Sort](https://leetcode.com/problems/parallel-courses-iii/) | Linear-time in DAGs | `dp[v] = min(dp[u] + weight(u,v))` | O(V+E) |
| **Shortest Path Cyclic (SPC)** | [Bellman-Ford](https://leetcode.com/problems/network-delay-time/), [Negative Weights](https://www.geeksforgeeks.org/bellman-ford-algorithm-dp-23/) | Handles cycles | `dp[v] = min(dp[v], dp[u] + weight(u,v))` | O(V·E) |
| **Process Scheduling (SPT)** | [Shortest Job First](https://www.geeksforgeeks.org/program-for-shortest-job-first-or-sjf-cpu-scheduling-set-1-non-preemptive/), [CPU Scheduling](https://leetcode.com/discuss/interview-question/operational/scheduling) | Minimize wait time | DP with time states | O(n log n) |
| **Transportation (TRANSPO)** | [Minimum Cost Flow](https://www.geeksforgeeks.org/minimum-cost-maximum-flow-from-a-graph-using-bellman-ford-algorithm/), [Shipping](https://leetcode.com/discuss/interview-question/operational/transportation) | Supply-demand matching | `dp[s][d] = min(cost + dp[s-supply][d-demand])` | O(S·D) |
| **Traveling Salesman (TSP)** | [Classic TSP](https://leetcode.com/problems/shortest-path-visiting-all-nodes/), [Hamiltonian Path](https://www.geeksforgeeks.org/travelling-salesman-problem-set-1/) | Minimal tour | `dp[mask][i] = min(dp[mask][i], dist[j][i] + dp[mask^(1<<i)][j])` | O(n²·2ⁿ) |



**Books && Videos && Blogs**\
[Dynamic Programming: A Computational Tool by Art Lew (Author), Holger Mauch (Author)](https://a.co/d/44vZOY5) \
[MIT Lecture 19: Dynamic Programming I: Fibonacci, Shortest Paths Instructor: Erik Demaine](https://ocw.mit.edu/courses/6-006-introduction-to-algorithms-fall-2011/resources/lecture-19-dynamic-programming-i-fibonacci-shortest-paths/) \
[MIT Notes Dynamic Programming](https://web.mit.edu/15.053/www/AMP-Chapter-11.pdf)
