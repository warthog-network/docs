---
label: Overview
---
# DeFi in Warthog

## Hard-Coded DeFi
Warthog implements a novel approach to Decentraled Finance (DeFi) by hard-coding the underlying logic directly into the node instead of relying on smart contract implementations. This approach has several crucual advantages over traditional DeFi implementatios:

### 1. Security

Today's DeFi platforms are constantly plagued by security vulnerabilities which are exploited by hackers. Such disastrous events regularly wipe liquidity pools and leave investors with significant losses. In consequence, DeFi could not gain as much trust as it deserves. Warthog is about to change this problem by including dedicated DeFi logic in its core. Native code is much easier to write and to check for vulnerabilities such that it can be expected that our approach will have few to no critical bugs impacting security.

### 2. Feature Support
Compared to smart contract based DeFi implementations, a hard-coded approach allows the impelmentation of novel features. For example, Warthog's revolutionary sandwich-free order matching algorithm simply not possible to implement without native node support because it treats all order jointly, erasing their implicit order of appearance within a block. A smart contract based approach is not capable of joint processing of transactions but instead requiers an ordering.

### 3. First-Class-Citizen
Warthog will be a crypto platform built with specific focus in DeFi. The entire database architecture is revolved around this purpose and therefore, insight and convenience functionality will be supported out of the box. This includes richlist for every token, easy-to-understand fork hierarchy and transparent tokenomics which don't require to trust dubious meme coin website charts.

## Unique Features
As elaborated above, in line with Warthog's spirit to push the boundaries of what is possible in blockchain technology, we are exploring new DeFi features not found in any other project:

### Sandwich-free DeFi
### Forking existing token distributions
By using a lazy copy-on-write database and careful design it is possible to allow forking existing balance distributions when creating new assets.
### Different Asset Creation Modes


## Details

### Tokens and Assets
Due to technical reasons we must distinguish between tokens and assets. Every asset is a token but there are also liquidity tokens and the WART token which are not considered assets. More precisely, tokens consist of three token types with the following properties:

Token Type   | Transferable | Tradable for | Redeemable
---    | --- | --- | ---
WART | yes | any asset | no
Assets | yes| WART | no
Liquidity Tokens | yes | none | yes

### Asset Creation
New Assets can be created by warthog users. During this process the initial distribution can be chosen by selecting an asset creation mode. The tokenomics overview will automatically be created based on this process and is unfakable.

#### Asset Names
Asset names consist of alphanumeric characters (`[A-Za-z0-9]`) of maximal length 5.

#### Asset Creation Modes
Later, Warthog will offer several differnt asset creation modes, however in the first phase we will only support traditional creation of an asset where the creator holds the entire supply.

### Trading
On Warthog every asset automatically has a single liquidity pool attached with WART as quote currency. Similarly to other DeFi platforms, traders will be able to provide liquidity to a pool in exchange for liquidity tokens, thus participating passively on trading activity, or actively create limit orders that will be automatically matched against other orders and against the liquidity pool.

This means that every asset can only be exchanged for WART while WART can be exchanged for any asset. Liquidity tokens cannot be traded but redeemed. Like in other DeFi implementations, depending on market conditions, holders of liquidity tokens can receive yield based on active pool trading of other users but can also suffer from impermanent loss when the price drifts away into one direction.

