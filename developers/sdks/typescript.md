# SDK for TypeScript
Warthog provides an easy-to-use SDK for TypeScript called [`warthog-ts`](https://www.npmjs.com/package/warthog-ts) intended for interacting with the Warthog ecosystem.

## Installation
```bash
npm install warthog-ts
```

## Features

- Generate wallets and sign transactions
- Built-in API client for node communication
- Works in Node.js, React Native, and Browsers

## Two node families

The network runs two families of node, and they are separate APIs:

| | Legacy node (`core/master`, `core/dev`) | DeFi node (`core/defi`) |
|---|---|---|
| Client | `WarthogLegacyApi` | `WarthogDeFiApi` |
| Chain head | flat | nested under `chainHead` |
| Transfer amount | `amountE8` | `wartE8` |
| Transaction types | 1, a transfer | 9 |
| Balances | the confirmed balance | total, locked, mempool |

Each family has its own client and its own transaction context, so the
compiler stops you from building something the node would reject. A legacy
context has one builder, `transferWart`, because that node accepts nothing
else, and it is the only one that can be sent to a legacy node.

## Quick Start

```typescript
import { Account, NonceId, RoundedFee, Wart, WarthogDeFiApi } from 'warthog-ts';

// 1. Connect to a node
const api = new WarthogDeFiApi('http://localhost:3100');

// 2. Load an account
const account = Account.fromPrivateKeyHex(privateKeyHex);

// 3. Create a transaction context (fetches chain pin automatically)
const context = await api.createTransactionContext(RoundedFee.min(), NonceId.random());

// 4. Build and sign a transaction
const tx = context.transferWart(account, toAddress, Wart.fromE8(100000000n)!);

// 5. Submit to node
const result = await api.submitTransaction(tx);
```

## The nonce

The nonce is a replay tag, not a counter. A transaction is identified by its
`(account, pin height, nonce)` triple, and the only check a node makes is
that it has not seen that triple before: a repeat is rejected with `ENONCE`,
unless the new transaction pays a higher fee, in which case it replaces the
old one.

The nonce does not have to increase, and no node reports an account's next
nonce, so there is nothing to fetch and nothing to keep track of. Use the
random one:

```typescript
import { NonceId } from 'warthog-ts';

const nonce = NonceId.random();
```

## Balances

Each node family reports balances its own way, so each client has its own
balance methods.

```typescript
// The legacy node reports one number: the confirmed balance.
const confirmed = await new WarthogLegacyApi(url).getWartBalance(address);
confirmed.E8;   // bigint

// The DeFi node splits it three ways.
const balance = await new WarthogDeFiApi(url).getWartBalance(address);
balance.total.E8;      // what the account owns
balance.locked.E8;     // locked in open limit orders
balance.mempool.E8;    // locked by transactions in the mempool
balance.free().E8;     // what can be spent now

// Token balances are the DeFi node's alone.
const tokenBalance = await new WarthogDeFiApi(url).getTokenBalance(address, assetHash);
tokenBalance.name;         // token name
tokenBalance.decimals;     // the token's TokenDecimals
tokenBalance.total.amount; // what the account owns
tokenBalance.free().amount; // what can be spent now
```
