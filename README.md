# Maximizing Cyclic Arbitrage Revenue Across Multiple Cycles in Decentralized Exchanges

## Background and Motivation

Decentralized exchanges (DEXs) are key infrastructures in the blockchain ecosystem, enabling direct cryptocurrency exchanges without intermediaries through Automated Market Maker (AMM) mechanisms. The AMM determines exchange rates based on liquidity reserves, causing a phenomenon known as slippage
where rates fluctuate dynamically with transaction volume. These rapid fluctuations create cyclic arbitrage opportunities — a searcher can exchange
cryptocurrencies through a cycle (e.g., A→B→C→A) to exploit transient price discrepancies and profit by increasing their holdings of the target cryptocurrency. Existing methods follow a "One-Detected, One-Execute" approach: they immediately execute arbitrage upon detecting a single profitable cycle, then repeat. This ignores how intermediate transactions alter liquidity reserves and how overlapping cycles interact, leaving significant profit on the table.

## The Problem of Existing Methods

Existing cyclic arbitrage methods follow a "One-Detected, One-Execute" approach — they detect a single profitable cycle, execute it immediately, and repeat. This strategy has three fundamental shortcomings:

### Challenge 1: Multi-Cycle Joint Optimization

In practice, arbitrage cycles frequently overlap, sharing one or more liquidity pools. Executing one cycle changes the reserves (and therefore the exchange rates) of shared pools, directly affecting the profitability of other cycles. Existing methods such as those that focus on single cycles or multiple independent cycles ignore these interdependencies.

### Challenge 2: Lack of Algorithm Guarantees

Many existing methods rely on heuristic or black-box optimization techniques that provide no theoretical guarantees on how close their solutions are to the optimal.

### Challenge 3: Computational Efficiency

Arbitrage opportunities are transient and must be exploited within tight time windows (a single block). Algorithms whose running time grows exponentially with input size are impractical.

## Problem Definition

The paper formally defines the **Arbitrage Optimization (AO) problem** as follows:

Given a set of potentially overlapping arbitrage cycles and an initial amount of a target cryptocurrency, the goal is to determine an investment strategy that maximizes the total output amount of the target cryptocurrency. The key constraint is that the total investment across all cycles cannot exceed the initial amount. Importantly, the problem is state-dependent: executing trades on one cycle changes the liquidity reserves of shared pools, which in turn affects the output of other cycles.

The AO problem is proven to be **NP-hard** via a reduction from the Partition problem, meaning no polynomial-time exact algorithm exists (assuming P != NP).

## Proposed Algorithms

### APS: Arbitrage Progressive Strategy

**Key idea:** Greedy progressive allocation. APS iteratively allocates the smallest budget unit (one atomic token unit) to whichever cycle currently offers the highest marginal gain, updating all pool states after each allocation.

**Theoretical properties:**
- **Approximation ratio: e/(e-1), approximately 1.58.** This guarantee is derived from the fact that the arbitrage revenue function is monotone submodular under a cardinality constraint, and the greedy algorithm is known to achieve this ratio for such problems.
- **Time complexity: O(M_in * n)**, where M_in is the input amount (in atomic units) and n is the number of cycles. This is pseudo-polynomial — it depends on the numeric value of the input, not just the problem size.
- **Best suited for:** scenarios with small input amounts where fast decisions are needed.
- In practice, the algorithm terminates early when no profitable cycle remains, making it faster than the worst-case bound suggests.

### ADP: Arbitrage Dynamic Programming

**Key idea:** Segment decomposition combined with dynamic programming. ADP decomposes the network of interleaved cycles into a sequence of linear segments (each containing multiple independent parallel paths), then optimizes allocation within each segment using DP, and composes the local solutions into a global strategy.

**Theoretical properties:**
- **Approximation ratio: e, approximately 2.72.** In practice, with a small approximation parameter epsilon, the actual ratio is much better than this theoretical upper bound.
- **Time complexity: O(n^4 / epsilon^2)**, where n is the number of cycles and epsilon is the approximation parameter. Crucially, this is **independent of M_in**, making ADP scalable to arbitrarily large input amounts.
- **Best suited for:** scenarios with large input amounts and dense cycle networks where APS's pseudo-polynomial complexity becomes prohibitive.

### Algorithm Comparison

| Property | APS | ADP |
|----------|-----|-----|
| Strategy | Global greedy, one unit at a time | Segment decomposition + local DP |
| Time complexity | O(M_in * n) — pseudo-polynomial | O(n^4 / epsilon^2) — polynomial |
| Approximation ratio | e/(e-1) ~ 1.58 (tighter) | e ~ 2.72 (looser bound) |
| Depends on input size | Yes | No |
| Ideal scenario | Small inputs, few cycles | Large inputs, dense networks |

### Baselines

- **ODE (One-Detected, One-Execute):** Identifies the single most profitable cycle and optimizes it independently, ignoring all other cycles.
- **MIC (Maximum Independent Cycles):** Greedily selects a set of non-overlapping (independent) cycles and optimizes each one separately, avoiding the complexity of shared liquidity pools but missing joint optimization opportunities.

## Demo

Visit the [online demo](https://maximizing-cyclic-arbitrage-revenue.netlify.app/#/) to interactively explore and compare the algorithms on sample arbitrage scenarios.
