---
label: Hard-Coded DeFi
---

# Hard-Coded DeFi
Warthog implements a novel approach to Decentralized Finance (DeFi) by hard-coding the underlying logic directly into the node instead of relying on smart contract implementations. This approach has several crucial advantages over traditional DeFi implementations:

1. **Security** Today's DeFi platforms are constantly plagued by security vulnerabilities which are exploited by hackers. Such disastrous events regularly wipe liquidity pools and leave investors with significant losses. In consequence, DeFi could not gain as much trust as it deserves. Warthog changes this by including dedicated DeFi logic in its core. Native code is much easier to write and to check for vulnerabilities such that our approach will have few to no critical bugs impacting security.

2. **Feature Support** Compared to smart contract based DeFi implementations, a hard-coded approach allows the implementation of novel features. For example, Warthog's revolutionary sandwich-free order matching algorithm is simply not possible to implement without native node support because it treats all orders jointly, erasing their implicit order of appearance within a block. A smart contract based approach is not capable of joint processing of transactions but instead requires an ordering.

3. **First-Class Citizen** Warthog is a crypto platform built with specific focus on DeFi. The entire database architecture is centered around this purpose and therefore, insight and convenience functionality is supported out of the box. This includes richlists for every token, easy-to-understand fork hierarchy and transparent tokenomics which don't require trusting dubious meme coin website charts.

# Unique Features
In line with Warthog's spirit to push the boundaries of what is possible in blockchain technology, we implement new DeFi features not found in any other project:

## MEV Protection by Design
Traditional DeFi protocols execute swap orders one at a time against a liquidity pool. This sequential execution means the pool state changes after each individual swap, creating an ordering dependency that block builders can exploit. By observing pending transactions in the mempool, attackers can position their own orders before or after a victim's order to extract value through sandwich attacks.

Warthog's Fair Batch Matching algorithm processes all swap orders within a block jointly rather than sequentially. This eliminates the possibility of ordering-based MEV extraction entirely. All buyers receive the same purchase price and all sellers receive the same selling price, making it impossible to profit at another user's expense through transaction reordering.

For more details on how Fair Batch Matching works, see the [Fair Batch Matching algorithm](fair-batch-matching.md) documentation.

## Forking Existing Token Distributions
!!!
This is a feature that is still in development.
!!!
By using a lazy copy-on-write database and careful design, it is possible to fork existing balance distributions when creating new assets. When a new token is created this way, all holders of the original token receive an identical balance of the new token. This enables use cases such as creating new tokens with balance distribution cloned from an existing token and paying out dividends proportionally to the holdings at a specific point in time.

The copy-on-write approach ensures efficiency: storage is only consumed when a cloned token holder makes their first transaction, diverging from the original distribution.

<!-- ### Different Asset Creation Modes -->
<!-- Warthog supports multiple asset creation modes for different distribution needs. Currently, the traditional mode creates an asset where the creator holds the entire initial supply. Future modes will enable fair auction distributions and other mechanisms to distribute the initial supply across multiple participants. -->
<!--  -->
<!-- ## Details -->
<!--  -->
<!-- ### Tokens and Assets -->
<!-- Due to technical reasons we must distinguish between tokens and assets. Every asset is a token but there are also liquidity tokens and the WART token which are not considered assets. More precisely, tokens consist of three token types with the following properties: -->
<!--  -->
<!-- | Token Type | Transferable | Tradable for | Redeemable | -->
<!-- | --- | --- | --- | --- | -->
<!-- | WART | yes | any asset | no | -->
<!-- | Assets | yes | WART | no | -->
<!-- | Liquidity Tokens | yes | none | yes | -->
<!--  -->
<!-- ### Asset Creation -->
<!-- New assets can be created by Warthog users. During this process the initial distribution is determined by the selected asset creation mode. The tokenomics overview is automatically created based on this process and is unfakable. -->
<!--  -->
<!-- #### Asset Names -->
<!-- Asset names consist of alphanumeric characters (`[A-Za-z0-9]`) of maximal length 5. -->
<!--  -->
<!-- #### Asset Creation Modes -->
<!-- Currently, only the traditional asset creation mode is supported, where the creator holds the entire initial supply. Future versions will offer additional modes for different distribution mechanisms. -->
<!--  -->
<!-- ### Trading -->
<!-- On Warthog every asset automatically has a single liquidity pool attached with WART as quote currency. Similarly to other DeFi platforms, traders can provide liquidity to a pool in exchange for liquidity tokens, thus participating passively on trading activity, or create limit orders that will be automatically matched against other orders and against the liquidity pool. -->
<!--  -->
<!-- This means that every asset can only be exchanged for WART while WART can be exchanged for any asset. Liquidity tokens cannot be traded but can be redeemed for their underlying assets. Like in other DeFi implementations, depending on market conditions, holders of liquidity tokens can receive yield based on active pool trading of other users but can also suffer from impermanent loss when the price drifts away in one direction. -->
<!--  -->
<!-- ## Links -->
<!-- - [Fair Batch Matching Paper (PDF)](https://github.com/CoinFuMasterShifu/FairBatchMatching/blob/main/FairBatchMatching.pdf) -->
<!-- - [DeFi Demo](https://warthog.network/defi-demo) -->
