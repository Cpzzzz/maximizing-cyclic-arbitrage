# Maximizing Cyclic Arbitrage Revenue Across Multiple Cycles in Decentralized Exchanges

## Background

Decentralized exchanges (DEXs) are key infrastructures in the blockchain ecosystem, enabling direct cryptocurrency exchanges without intermediaries through Automated Market Maker (AMM) mechanisms. The AMM determines exchange rates based on liquidity reserves, causing a phenomenon known as slippage where rates fluctuate dynamically with transaction. These rapid fluctuations create cyclic arbitrage opportunities — a searcher can exchange cryptocurrencies through a cycle (e.g., $A \to B \to C \to A$) to exploit transient price discrepancies and profit by increasing their holdings of the target cryptocurrency. 

Existing cyclic arbitrage methods follow a "One-Detected, One-Execute" approach — they detect a single profitable cycle, execute it immediately, and repeat. This strategy has three fundamental shortcomings: (1) Multi-Cycle Joint Optimization: cycle overlapping is prevalent, yet existing works focus on single cycles or multiple independent cycles, neglecting higher revenue from joint optimization; (2) Lack of Algorithm Guarantees: existing heuristic and black-box methods provide no theoretical guarantees on how close the solution is to optimal; (3) Computational Efficiency: arbitrage opportunities are transient and must be exploited within a block time, yet existing methods do not adequately control computational complexity.

## Problem Definition

To address the above challenges, this paper formally defines the **Arbitrage Optimization (AO) problem** as follows:

Given a set of potentially overlapping arbitrage cycles and an initial amount of a target cryptocurrency, the goal is to determine an investment strategy that maximizes the total output amount of the target cryptocurrency. The key constraint is that the total investment across all cycles cannot exceed the initial amount. Importantly, the problem is state-dependent: executing trades on one cycle changes the liquidity reserves of shared pools, which in turn affects the output of other cycles.

The AO problem is proven to be **NP-hard** via a reduction from the Partition problem, meaning no polynomial-time exact algorithm exists (assuming $P \neq NP$).

## Proposed Algorithms

### APS: Arbitrage Progressive Strategy

**Key idea:** Greedy progressive allocation. APS iteratively allocates the smallest budget unit (one atomic token unit) to whichever cycle currently offers the highest marginal gain, updating all pool states after each allocation.

**Theoretical properties:**
- **Approximation ratio:** $\frac{e}{e-1} \approx 1.58$. This guarantee is derived from the fact that the arbitrage revenue function is monotone submodular under a cardinality constraint, and the greedy algorithm is known to achieve this ratio for such problems.
- **Time complexity:** $O(M_{in} \cdot n)$, where $M_{in}$ is the input amount (in atomic units) and $n$ is the number of cycles. This is pseudo-polynomial — it depends on the numeric value of the input, not just the problem size.
- **Best suited for:** scenarios with small input amounts where fast decisions are needed.
- In practice, the algorithm terminates early when no profitable cycle remains, making it faster than the worst-case bound suggests.

### ADP: Arbitrage Dynamic Programming

**Key idea:** Segment decomposition combined with dynamic programming. ADP decomposes the network of interleaved cycles into a sequence of linear segments (each containing multiple parallel paths), then optimizes allocation within each segment using DP, and composes the local solutions into a global strategy.

**Theoretical properties:**
- **Approximation ratio:** $e \approx 2.72$. In practice, with a small approximation parameter $\epsilon$, the actual ratio is much better than this theoretical upper bound.
- **Time complexity:** $O(\frac{n^4}{\epsilon^2})$, where $n$ is the number of cycles and $\epsilon$ is the approximation parameter. Crucially, this is **independent of $M_{in}$**, making ADP scalable to arbitrarily large input amounts.
- **Best suited for:** scenarios with large input amounts and dense cycle networks where APS's pseudo-polynomial complexity becomes prohibitive.

### Algorithm Comparison

| Property | APS | ADP |
|----------|-----|-----|
| Strategy | Global greedy, one unit at a time | Segment decomposition + local DP |
| Time complexity | $O(M_{in} \cdot n)$ — pseudo-polynomial | $O(\frac{n^4}{\epsilon^2})$ — polynomial |
| Approximation ratio | $\frac{e}{e-1} \approx 1.58$ (tighter) | $e \approx 2.72$ (looser bound) |
| Depends on input size | Yes | No |
| Ideal scenario | Small inputs, few cycles | Large inputs, dense networks |

### Baselines

- **ODE (One-Detected, One-Execute):** Identifies the single profitable cycle and uses the gradually increasing linear search to maximize the revenue, then repeats the process.
- **MIC (Multiple Independent Cycles):** Greedily selects a set of non-overlapping (independent) cycles and optimizes each one separately, avoiding the complexity of shared liquidity pools but missing joint optimization revenue.

## Demo

Visit the [online demo](https://maximizing-cyclic-arbitrage-revenue.netlify.app/#/) to interactively explore and compare the algorithms on sample arbitrage scenarios.
