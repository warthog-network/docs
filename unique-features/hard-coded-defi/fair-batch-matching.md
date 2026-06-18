# Fair Batch Matching

Warthog takes an innovative approach to DeFi in the sense that a hybrid setting of two liquidity sources (pool and order book) is considered and addressed mathematically optimally. Our [scientific research](https://github.com/CoinFuMasterShifu/FairBatchMatching/blob/main/FairBatchMatching.pdf) first describes this setup and proves existence and uniqueness of an optimal and fair order matching which operates batch-wise jointly on all orders such that the same price is realized for all buy swaps and all sell swaps. In particular no sandwich attacks are possible by design.

This article is a high-level guide to how Fair Batch Matching works in practice on the Warthog DEX. It complements the [full technical paper](https://github.com/CoinFuMasterShifu/FairBatchMatching/blob/main/FairBatchMatching.pdf) by giving you the practical mental model and answering the questions that come up most often. Try it yourself at [warthog.network/defi-demo](https://warthog.network/defi-demo).

## The problem with sequential matching

In traditional DeFi protocols, swap orders are matched against liquidity pools one at a time. Each transaction interacts with the pool sequentially, which means the pool state changes after every individual swap. This creates an ordering dependency that block builders can exploit: by observing pending transactions in the mempool, they can position their own orders before or after a victim's order to extract value. This practice, known as MEV extraction, manifests most commonly as sandwich attacks where a victim's trade is surrounded by attacker transactions that buy low and sell high at the victim's expense.

Warthog's DEX takes a fundamentally different approach: it processes all swap orders within a block *jointly*, finding a single price at which the matching is fair to every participant. This completely eliminates the possibility of ordering-based MEV extraction: the same conversion price is shared for all buy liquidity and all sell liquidity, so there is nothing for an attacker to gain by reordering their own orders around a victim's.

## Two kinds of liquidity

In Warthog's DEX, every asset market has two kinds of liquidity available for matching:

- **Continuous liquidity** provided by a liquidity pool. The pool holds reserves of the base asset and the quote asset (WART) and follows the standard constant-product market maker formula. This liquidity is continuous because the cumulative liquidity is a continuous function of the price drift.
- **Discrete liquidity** in the form of an order book of limit orders. Each order has a fixed limit price and quantity. This liquidity is discrete because it is concentrated at these price levels.

Traditional exchanges only support discrete liquidity while today's DeFi's automated market makers such as Uniswap V1-V4 only support continuous liquidity. Warthog is the world's first and only place to unify both liquidity concepts and offer a theoretically sound matching algorithm in order to approach this hybrid setting with mathematical rigour, game-theoretic fairness and algorithmic robustness.

## Liquidity Pools

A liquidity pool holds reserves of two assets, the asset we measure the price of is the *base asset* whereas the asset we express the price in is the *quote asset*. In Warthog the quote asset is always WART, i.e. all prices are expressed in WART and no direct trades between non-WART tokens are possible. Every swap happens against WART.

The pool operates on a simple rule: when exchanging assets with the pool, the product of the two reserve amounts must remain constant. This is called the Constant Product Market Maker (CPMM) formula, also used by Uniswap.

For example, if a pool has 1000 base tokens and 100 WART, the product is 100,000. When someone buys base tokens, they deposit WART and receive base tokens. Ignoring fees for now, the pool adjusts both reserves so the product stays 100,000. The more base tokens someone buys, the higher the price they pay — a natural price discovery mechanism.

## Order Books

An order book consists of swap orders, where each order has two components. There's a **limit price** — the worst price the trader is willing to accept — and a **quantity** — how much they want to trade.

There are two types of orders. **Base swap orders** (sell orders) swap base into quote token, meaning you sell the base asset for WART. **Quote swap orders** (buy orders) swap quote into base token, meaning you buy the base asset with WART.

In Warthog, the amount to be swapped is always measured in the input currency. This differs from traditional order books where amounts are measured in the base currency. For sell orders the behavior is identical, but for buy orders, WART orders specify how much WART to spend while traditional order books specify the amount of base asset to buy.

No duplicate price levels are allowed within either base or quote swap orders but there can exist pairs of base and quote swap orders at the same price. This restriction is for simplicity in the matching setting, as otherwise matching priority at the same price level would have to be defined. The Warthog node bypasses this restriction by aggregating orders by price level before computing a Fair Batch Matching — see [Same Price Orders](#same-price-orders) below for details.

When looking at a specific price point, we can determine which orders are filled. Sell orders with limit prices below that price are filled because someone is willing to sell cheap. Buy orders with limit prices above that price are filled because someone is willing to buy expensive. Orders on the other side of that price are not filled. At the same price level, only one side can be partially filled — no partial fills for both buyers and sellers.

The total amount being bought and sold at a given price is called the fill sum.

## Pool Interaction

Since Warthog allows orders to interact with both the other side of the order book and the liquidity pool, rules are needed for how this works. When buyers want to purchase base assets, they can either exchange directly with other traders who are selling through the order book, or exchange with the liquidity pool. The same applies to sellers.

The key question is: how much of the trading happens at the pool versus through direct exchange?

## Fair Pool Interaction

If the price at the liquidity pool is more favorable for buyers or sellers compared to the order book, they prefer using the liquidity pool. A pool interaction is considered fair when both buyers and sellers have offloaded the optimal amount of liquidity to the pool — meaning neither party prefers changing the amount of directly order-matched and pool-traded liquidity to improve their conversion rate.

A fair interaction can be viewed as a Nash equilibrium, a state where no participant can get a better deal by unilaterally changing their strategy.

## Fair Batch Matching: The Only Fair Matching

Warthog's custom matching engine implements *Fair Batch Matching* (FBM), a mathematical framework that batch processes all swap orders within a block jointly rather than sequentially. FBM is the *only fair way* to match liquidity in the above hybrid setting of continuous and discrete liquidity. Every participant is satisfied with this matching, and there is no remaining pair of unfilled buy and sell liquidity that could be matched against each other.

This approach completely eliminates the possibility of ordering-based MEV extraction because the same conversion price is shared for all buy liquidity and all sell liquidity such that there is nothing for an attacker to gain by reordering their own orders around a victim's.

### The Fair Batch Matching Theorem

A matching describes the amount of liquidity that buyers and sellers spend on pool interaction and filling the other party's limit orders, which implicitly defines an average conversion price. A Fair Batch Matching is a matching with fair pool interaction.

The key mathematical result of FBM is that for any order book and liquidity pool configuration, there always exists a unique Fair Batch Matching. This means a Fair Batch Matching always exists, and that matching is the only one that can satisfy these conditions.

This theorem is crucial because it allows defining a fair price and guarantees that the Fair Batch Matching algorithm implemented in Warthog always finds a well-defined result.

Warthog is the first cryptocurrency project to formulate this result and also the only project to have successfully implemented an optimization algorithm for practical use. Technically interested readers can find the implementation of this matching algorithm [here](https://github.com/warthog-network/core/tree/defi/src/shared/src/defi).

With Fair Batch Matching no actor can do strictly better by changing how they interact with the pool or the order book and also there is no remaining pair of unfilled buy and sell liquidity that could be matched against each other, in the sense that any further matching would require either violating an order's limit price or making some actor worse off. In particular Fair Batch Matching is the Nash equilibrium.

## How Matching Works in Practice

When a block is processed, the node collects all pending limit swap orders for each market and looks at the current liquidity pool state. The FBM algorithm then determines a single price at which all trades execute, how much each order gets filled, and how much happens through the pool versus direct order matching. The node executes all matches atomically, and the pool state updates once with the combined result.

This means a victim's order cannot be sandwiched because there is no before and after — all orders execute simultaneously at the fair price.

## A mental model: selfish actors wanting the best conversion rate

The mathematical definition of FBM can be hard to internalize. Here is a more intuitive way to think about it, in the spirit of how the theory was developed.

Each side of the market is driven by individual *selfish actors* that want to achieve the best conversion rate. Liquidity from the order book can be

1. *unmatched*,
2. matched with the *other side of the order book*, or
3. matched with the *liquidity pool*.

If the liquidity pool is non-empty, i.e. has reserves of base and quote assets, it can be used by these actors as long as it offers a better conversion rate than matching with the other side of the order book.

### Iterative mental model

While we have implemented Fair Batch Matching as a bisection algorithm, for better understanding we can imagine it as an iterative process: for each matching constellation we can ask what selfish actors would do differently to achieve better conversion rate.

If the pool is currently offering a better price than the standing limit orders on the other side, liquidity will be partially offloaded to the pool. Conversely, if pool price is worse than the other side of the order book, actors will reduce the pool converted amount of funds and choose the liquidity from the order book. It can even be beneficial to partially match a single order and split the matched amount between the other side of the order book and the pool.

This leads to an iterative thought process where either buyers or sellers want to optimize their conversion rate until there is nothing that can be optimized. In that case we have found the *Nash equilibrium*, which we have shown is unique and which is the Fair Batch Matching.

#### Examples

In the iterative mental model each side picks the better deal for itself.

- If the pool price is better for buyers than the standing sell orders, buyers go to the pool until pool price reaches the best sell order.
- If the pool price is better for sellers than the standing buy orders, sellers go to the pool until pool price reaches the best buy order.
- Once the pool price matches a limit order, that order takes over for additional liquidity on that side.
- If a trade is larger than the pool can provide before its price would cross a limit order, the part that fits in the pool is filled there, the rest matches against the order book.
- Low-reserve pools move price quickly with each trade, so partial pool + partial order book fills are common in those markets.
- In a market with no limit orders on one side, all of that side's flow goes through the pool.
- In a market with no pool yet, all trading is direct order book matching.

## Counter-Order Paradox

A frequent surprise: a buy order and a sell order are placed at same price but the sell is unfilled, while the buy order is reported as fully filled. What happened?

The order on the other side of the order book was not filled in the way one would intuitively expect because of the presence of the liquidity pool. If the pool price is not picked to be exactly equal to the order price, it is beneficial for one side to swap liquidity at the pool rather than match with the other side of the order book such that the order's counter is not addressed. This explains the observed behavior.

If a user wants to force a match against the order book rather than the pool, they need to set a limit price that crosses the pool price enough to make matching against their order the better choice for the other side.

## Same Price Orders

In the FBM setting we do not allow multiple orders at the same price. The Warthog node improves this shortcoming by aggregating orders that share a price level into a single effective entry per price before running FBM, then post-processing to distribute the filled amount back to the individual orders.

### Price Level Aggregation

In order to support same price orders, the node proceeds by, before matching, grouping orders that share a price level, summing their amounts into a single effective entry per price, running the FBM algorithm on the aggregated levels, and then post-processing to figure out which individual orders were actually filled.

### Fill Priority

When multiple orders share the same price level and the matched liquidity is less than the total amount of all orders at this price level, the node has to decide which of these orders are filled. The *fill priority* used here is decreasing in the internal *order id*, which is itself increasing in block height. This means that orders mined in earlier blocks have higher priority. Within a single block, the miner can arrange the order in which transactions are included, which means the miner has control over the fill priority of same-price orders in that block.

However, since all buyers and all sellers receive the same price, this priority only determines whether an order is matched at all but not the matching price.

## Pool Mechanics

Every asset automatically receives a WART liquidity pool as quote currency. The pool follows the standard constant-product market maker formula (the same math Uniswap uses): when exchanging assets with the pool, the product of the two reserve amounts stays constant. The current price (the marginal conversion rate for an infinitesimally small trade) is the quotient of the two reserve amounts: if the pool holds $B$ units of base and $Q$ units of WART, the current price is $Q / B$ (WART per unit of base). There are no upper or lower price boundaries set on the pool. A pool with very low reserves moves price dramatically on each trade; a deep pool is relatively stable.

Each swap through a pool pays a small fee. The current default in the testnet is *0.10% (10 basis points)*. The fee is retained in the pool rather than sent to a separate recipient. After each swap, the product of the two reserve amounts does not stay constant but instead *increases* slightly, by the fee. Over time, as more trades occur, the pool's reserves grow beyond what would be expected from a pure constant-product model. This benefits the liquidity providers, who own shares of the pool proportional to their contribution. When they withdraw, they receive their share of the (now larger) pool.

In other words: the fee is paid by traders and accumulated by the pool. It is not extracted to any external address. Traders pay a small fee, liquidity providers earn that fee proportional to their share, and the pool itself grows over time.

## Key Benefits

- **No MEV extraction**: Transaction ordering cannot be exploited for profit
- **Equal pricing**: All participants trading at the same time receive identical prices
- **Simplicity**: No need for encrypted mempools or complex ordering schemes to protect orders from MEV extractors
- **Nash equilibrium**: Fair outcomes for all participants by construction

## Links

- [Fair Batch Matching Paper (PDF)](https://github.com/CoinFuMasterShifu/FairBatchMatching/blob/main/FairBatchMatching.pdf)
- [DeFi Demo](https://warthog.network/defi-demo)
