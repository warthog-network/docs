# Fair Batch Matching

## Overview

Fair Batch Matching (FBM) is a mathematical framework for matching swap orders with liquidity pools in a way that eliminates transaction ordering manipulation. Unlike traditional DeFi where orders are processed one at a time sequentially, FBM processes all orders within a block jointly, ensuring every participant receives the same fair price regardless of transaction order.

This document explains how FBM works without mathematical equations. For the full mathematical treatment, see the [Fair Batch Matching paper](https://github.com/CoinFuMasterShifu/FairBatchMatching/blob/main/FairBatchMatching.pdf).

## Liquidity Pools

A liquidity pool is a smart contract that holds reserves of two assets - a base asset and a quote asset. In Warthog, the quote asset is always WART. This implies that no direct trades between non-WART tokens are possible, each swap happens against the native WART token.

The pool operates on a simple rule: when you exchange assets with the pool, the product of the two reserve amounts must remain constant. This is called the Constant Product Market Maker (CPMM) formula, also used by Uniswap.

For example, if a pool has 1000 base tokens and 100 WART, the product is 100,000. When someone buys base tokens, they deposit WART and receive base tokens. Ignoring any fees for now, the pool adjusts both reserves so their product stays 100,000. This means the more base tokens someone buys, the higher the price they pay - a natural price discovery mechanism. Requiring the product to be constant is just one way of defining a price curve for an automated market maker, however we do not consider different setups here.

## Order Books

An order book consists of swap orders, where each order has two components:
- A **limit price** - the worst price the trader is willing to accept
- A **quantity** - how much they want to trade

There are two types of orders:
- **Base swap orders** (sell orders) - swap base into quote token, i.e. sell the base asset for WART
- **Quote swap orders** (buy orders) - swap quote into base token, i.e. buy the base asset with WART

Note that the amount to be swapped is always measured in the input currency here, compared to traditional order books where the amount traded is always measured in the base currency. For sell orders the behavior is identical but for buy orders, in WART the order specifies how much WART to spend on the purchase while traditional order books specify the amount of base asset to buy.

When we look at a specific price point, we can determine which orders are filled:
- Sell orders with limit prices below that price are filled (someone willing to sell cheap)
- Buy orders with limit prices above that price are filled (someone willing to buy expensive)
- Orders on the other side of that price are not filled
- For the same price level we restrict that only one side can be partially filled (no partial fills for both, buyers and sellers).

The total amount being bought and sold at a given price is called the fill sum.

## Pool Interaction

Since Warthog allows orders to interact with both the other side of the order book and the liquidity pool, we need rules for how this works.

When buyers want to purchase base assets, they can either:
1. Exchange directly with other traders who are selling (through the order book)
2. Exchange with the liquidity pool

The same applies to sellers.

The key question is: how much of the trading happens at the pool versus through direct exchange?

## Fair Pool Interaction

If the price at the liquidity pool is more favorable for buyers or sellers compared to the order book, they prefer using the liquidity pool. A pool interaction is considered *fair* when both, buyers and sellers have offloaded the optimal amount of liquidity to the pool, i.e. when neither party prefers changing the amount of directly order-matched and pool-traded liquidity to improve their conversion rate.

A fair interaction can be viewed as a Nash equilibrium - a state where no participant can get a better deal by unilaterally changing their strategy.

## The Fair Batch Matching Theorem

A matching describes the amount of liquidity that buyers and sellers spend on pool interaction and filling the other party's limit orders, which implicitly defines an average conversion price. A Fair Batch Matching is a matching with fair pool interaction.

The key mathematical result of FBM is that for any order book and liquidity pool configuration, there always exists a **unique** Fair Batch Matching. This means:

1. **Existence**: A Fair Batch Matching always exists.
2. **Uniqueness**: That matching is the only one that can satisfy these conditions

This theorem is crucial because it allows to define a fair price and guarantees that the Fair Batch Matching algorithm we have implemented in Warthog always finds a well-defined result. 

We are the first cryptocurrency project that has even formulated this result and also the only project that has successfully implemented an optimization algorithm for practical use. Technically interested readers can find the implementation of this matching algorithm [here](https://github.com/warthog-network/core/tree/8ea077884bf325ab9495bd3ddebc90fa828bc051/src/shared/src/defi). There is also a demo [on the warthog website](https://warthog.network/defi-demo).

## Why Sandwich Attacks Fail

Traditional DeFi is vulnerable to sandwich attacks because:

1. Orders are processed one at a time against the pool
2. Each order changes the pool state (reserves)
3. An attacker can see a victim's pending order in the mempool
4. The attacker places one order before and one after the victim
5. The attacker profits at the victim's expense

For example, if a victim wants to buy 1000 units:
1. Attacker buys first at the current pool price
2. Victim's order now executes at a higher price (because attacker bought first)
3. Attacker sells back at the higher price, keeping the profit

With Fair Batch Matching, this attack becomes impossible:

1. All orders within a block are collected
2. The FBM algorithm finds the unique fair price
3. All buyers pay the same price
4. All sellers receive the same price
5. The attacker's front-run and back-run orders receive the same price as the victim
6. No profit can be extracted from ordering manipulation

The critical difference is that FBM processes orders **jointly** rather than **sequentially**. The ordering of transactions within a block becomes irrelevant.

## How Matching Works in Practice

When a block is processed:

1. The node collects all pending limit swap orders for each market
2. It looks at the current liquidity pool state
3. The FBM algorithm determines:
   - A single price at which all trades execute
   - How much each order gets filled
   - How much happens through the pool versus direct order matching
4. The node executes all matches atomically
5. The pool state updates once with the combined result

This means a victim's order cannot be "sandwiched" because there is no before and after - all orders execute simultaneously at the fair price.

## Key Benefits

- **No MEV extraction**: Transaction ordering cannot be exploited for profit
- **Equal pricing**: All participants trading at the same time receive identical prices
- **Simplicity**: No need for encrypted mempools or complex ordering schemes to protect orders from MEV extractors
- **Nash equilibrium**: Fair outcomes for all participants by construction

## Links
- [Fair Batch Matching Paper (PDF)](https://github.com/CoinFuMasterShifu/FairBatchMatching/blob/main/FairBatchMatching.pdf)
- [DeFi Demo](https://warthog.network/defi-demo)
