# Fair Batch Matching

*Fair Batch Matching* (FBM) is Warthog's defense against *Sandwich* and similar MEV attacks. While the [full technical paper](https://github.com/CoinFuMasterShifu/FairBatchMatching/blob/main/FairBatchMatching.pdf) covers the mathematical details, this article explains how FBM works in plain terms.

## Liquidity Pools

A liquidity pool is a smart contract that holds reserves of two assets, the asset we measure the price of is the *base asset* whereas the asset we express the price in is the *quote asset*. In Warthog the quote asset is always WART, i.e. all prices are expressed in WART and no direct trades between non-WART tokens are possible. Every swap happens against WART.

The pool operates on a simple rule: when exchanging assets with the pool, the product of the two reserve amounts must remain constant. This is called the Constant Product Market Maker (CPMM) formula, also used by Uniswap.

For example, if a pool has 1000 base tokens and 100 WART, the product is 100,000. When someone buys base tokens, they deposit WART and receive base tokens. Ignoring fees for now, the pool adjusts both reserves so the product stays 100,000. The more base tokens someone buys, the higher the price they pay - a natural price discovery mechanism.

## Order Books

An order book consists of swap orders, where each order has two components. There's a **limit price** - the worst price the trader is willing to accept - and a **quantity** - how much they want to trade.

There are two types of orders. **Base swap orders** (sell orders) swap base into quote token, meaning you sell the base asset for WART. **Quote swap orders** (buy orders) swap quote into base token, meaning you buy the base asset with WART.

In Warthog, the amount to be swapped is always measured in the input currency. This differs from traditional order books where amounts are measured in the base currency. For sell orders the behavior is identical, but for buy orders, WART orders specify how much WART to spend while traditional order books specify the amount of base asset to buy.

When looking at a specific price point, we can determine which orders are filled. Sell orders with limit prices below that price are filled because someone is willing to sell cheap. Buy orders with limit prices above that price are filled because someone is willing to buy expensive. Orders on the other side of that price are not filled. At the same price level, only one side can be partially filled - no partial fills for both buyers and sellers.

The total amount being bought and sold at a given price is called the fill sum.

## Pool Interaction

Since Warthog allows orders to interact with both the other side of the order book and the liquidity pool, rules are needed for how this works. When buyers want to purchase base assets, they can either exchange directly with other traders who are selling through the order book, or exchange with the liquidity pool. The same applies to sellers.

The key question is: how much of the trading happens at the pool versus through direct exchange?

## Fair Pool Interaction

If the price at the liquidity pool is more favorable for buyers or sellers compared to the order book, they prefer using the liquidity pool. A pool interaction is considered fair when both buyers and sellers have offloaded the optimal amount of liquidity to the pool - meaning neither party prefers changing the amount of directly order-matched and pool-traded liquidity to improve their conversion rate.

A fair interaction can be viewed as a Nash equilibrium, a state where no participant can get a better deal by unilaterally changing their strategy.

## The Fair Batch Matching Theorem

A matching describes the amount of liquidity that buyers and sellers spend on pool interaction and filling the other party's limit orders, which implicitly defines an average conversion price. A Fair Batch Matching is a matching with fair pool interaction.

The key mathematical result of FBM is that for any order book and liquidity pool configuration, there always exists a unique Fair Batch Matching. This means a Fair Batch Matching always exists, and that matching is the only one that can satisfy these conditions.

This theorem is crucial because it allows defining a fair price and guarantees that the Fair Batch Matching algorithm implemented in Warthog always finds a well-defined result.

Warthog is the first cryptocurrency project to formulate this result and also the only project to have successfully implemented an optimization algorithm for practical use. Technically interested readers can find the implementation of this matching algorithm [here](https://github.com/warthog-network/core/tree/8ea077884bf325ab9495bd3ddebc90fa828bc051/src/shared/src/defi). There is also a demo [on the warthog website](https://warthog.network/defi-demo).

## Why Sandwich Attacks Fail

In traditional DeFi, sandwich attacks work because orders are processed one at a time against the pool. Each order changes the pool state, and an attacker monitoring the mempool can see a victim's pending order. The attacker then places one order before and one after the victim, profiting at the victim's expense.

For example, if a victim wants to buy an asset, the attacker buys first at the current pool price. The victim's order then executes at a higher price because the attacker bought first. The attacker sells back at the higher price, keeping the profit.

With Fair Batch Matching, this attack becomes impossible. All orders within a block are collected, the FBM algorithm finds the unique fair price, all buyers pay the same price, and all sellers receive the same price. The attacker's front-run and back-run orders receive the same price as the victim, so no profit can be extracted from ordering manipulation.

The critical difference is that FBM processes orders jointly rather than sequentially. The ordering of transactions within a block becomes irrelevant. This is the huge advantage of hard-coding DeFi on consensus level rather than relying on second layer smart contract implementations.

## How Matching Works in Practice

When a block is processed, the node collects all pending limit swap orders for each market and looks at the current liquidity pool state. The FBM algorithm then determines a single price at which all trades execute, how much each order gets filled, and how much happens through the pool versus direct order matching. The node executes all matches atomically, and the pool state updates once with the combined result.

This means a victim's order cannot be sandwiched because there is no before and after - all orders execute simultaneously at the fair price.

## Key Benefits

- **No MEV extraction**: Transaction ordering cannot be exploited for profit
- **Equal pricing**: All participants trading at the same time receive identical prices
- **Simplicity**: No need for encrypted mempools or complex ordering schemes to protect orders from MEV extractors
- **Nash equilibrium**: Fair outcomes for all participants by construction

## Links

- [Fair Batch Matching Paper (PDF)](https://github.com/CoinFuMasterShifu/FairBatchMatching/blob/main/FairBatchMatching.pdf)
- [DeFi Demo](https://warthog.network/defi-demo)
