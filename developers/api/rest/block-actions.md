# Block Actions

!!!
This documentation was partially generated with AI augmentation. Report inconsistencies at [github.com/warthog-network/docs/issues](https://github.com/warthog-network/docs/issues).
!!!

Every block affects the chain state via actions. These actions are returned from API methods such as `/transaction/latest` and `/chain/block/:id`.

## Block Structure

The response hierarchy is:

```
ActionsByBlock = {
  perBlock: BlockEntry[],
  fromId: uint64_t
}
BlockEntry = {
  height: uint32_t,
  confirmations: uint32_t,
  actions: BlockActions
}
BlockActions = {
  reward: WithHistoryId<Transaction<RewardData>> | null,
  wartTransfers: WithHistoryId<SignedTransaction<WartTransferData>>[],
  tokenTransfers: WithHistoryId<SignedTransaction<TokenTransferData>>[],
  newOrders: WithHistoryId<SignedTransactionProcessed<NewOrderData, NewOrderProcessed>>[],
  matches: WithHistoryId<Transaction<MatchData>>[],
  liquidityDeposits: WithHistoryId<SignedTransactionProcessed<LiqDepositData, LiqDepositProcessed>>[],
  liquidityWithdrawals: WithHistoryId<SignedTransactionProcessed<LiqWithdrawData, LiqWithdrawProcessed>>[],
  assetCreations: WithHistoryId<SignedTransactionProcessed<AssetCreateData, AssetCreateProcessed>>[],
  cancelations: WithHistoryId<SignedTransactionProcessed<CancelData, CancelProcessed>>[]
}
```

## Transaction Categories

Every action entry follows the same wrapper pattern:

```json
{
  "historyId": 123,
  "transaction": {
    "data": { ... },
    "hash": "...",
    "signedCommon": { ... },
    "processed": { ... }
  }
}
```

The `data`, `hash`, `signedCommon`, and `processed` fields contain the action-specific information.

### Unsigned Transactions

Reward and Match are unsigned — they are the result of block production, not submitted by users. They have no `signedCommon`:

```json
{
  "historyId": 1,
  "transaction": {
    "data": { ... },
    "hash": "..."
  }
}
```

### Signed Transactions

Wart Transfer, Token Transfer, New Order, Liquidity Deposit, Liquidity Withdrawal, Asset Creation, and Cancelation are signed — they are submitted by users and include `signedCommon` and optionally `processed`:

```json
{
  "historyId": 2,
  "transaction": {
    "data": { ... },
    "hash": "...",
    "signedCommon": { ... },
    "processed": { ... }
  }
}
```

### Common Field Types

**TransactionSignedCommon** — present in all signed transactions:

| Field | Type | Description |
|-------|------|-------------|
| `fee` | `Wart` | Transaction fee |
| `nonceId` | uint32_t | Nonce ID |
| `originAddress` | string | Sender address |
| `originId` | uint64_t | Origin transaction ID |
| `pinHeight` | uint32_t | Pin block height |

**Wart** — used for WART coin amounts:

| Field | Type | Description |
|-------|------|-------------|
| `E8` | uint64_t | Amount in smallest unit (1 WART = 10^8 E8) |
| `str` | string | Human-readable decimal string |

**FundsDecimal** — used for token amounts:

| Field | Type | Description |
|-------|------|-------------|
| `u64` | uint64_t | Amount in smallest unit |
| `str` | string | Human-readable decimal string |
| `decimals` | int32_t | Token decimal places |

**AssetBasic:**

| Field | Type | Description |
|-------|------|-------------|
| `hash` | string | Asset hash |
| `id` | uint64_t | Asset ID |
| `name` | string | Asset name |
| `decimals` | uint32_t | Asset decimals |

**PriceDetail** — used in limit orders:

| Field | Type | Description |
|-------|------|-------------|
| `precExponent10` | int32_t | Base-10 precision exponent |
| `exponent2` | int32_t | Power-of-2 exponent |
| `mantissa` | int32_t | 16-bit mantissa |
| `hex` | string | Raw bytes (hex) |
| `doubleAdjusted` | double | Price as double (adjusted) |
| `doubleRaw` | double | Price as double (raw) |

**BaseQuote** — used for liquidity pools and swaps:

| Field | Type | Description |
|-------|------|-------------|
| `base` | `FundsDecimal` | Base asset amount |
| `quote` | `Wart` | Quote asset amount (WART) |

## Reward

Block reward paid to miners. This is an unsigned transaction — there is no `signedCommon`.

```json
{
  "historyId": 1,
  "transaction": {
    "data": {
      "amount": { "E8": 300000000, "str": "3.00000000" },
      "toAddress": "3661579d..."
    },
    "hash": "da3dcfbc0dcb60be..."
  }
}
```

| Field | Type | Description |
|-------|------|-------------|
| `historyId` | uint64_t | History entry ID |
| `transaction.data.amount` | `Wart` | Reward amount |
| `transaction.data.toAddress` | string | Recipient address |
| `transaction.hash` | string | Transaction hash |

## Wart Transfer

WART coin transfer between accounts. This is a signed transaction.

```json
{
  "historyId": 40,
  "transaction": {
    "data": {
      "amount": { "E8": 100000000, "str": "1.00000000" },
      "toAddress": "00000000..."
    },
    "hash": "897b3cc72950b41e...",
    "signedCommon": {
      "fee": { "E8": 9992, "str": "0.00009992" },
      "nonceId": 0,
      "originAddress": "3661579d...",
      "originId": 12344,
      "pinHeight": 0
    }
  }
}
```

| Field | Type | Description |
|-------|------|-------------|
| `historyId` | uint64_t | History entry ID |
| `transaction.data.amount` | `Wart` | Transfer amount |
| `transaction.data.toAddress` | string | Recipient address |
| `transaction.hash` | string | Transaction hash |
| `transaction.signedCommon` | object | Signing metadata |

## Token Transfer

Token transfer between accounts. Tokens may be standard assets or liquidity pool shares (`isLiquidity: true`). This is a signed transaction.

```json
{
  "historyId": 41,
  "transaction": {
    "data": {
      "amount": { "str": "99.9912", "u64": 999912, "decimals": 4 },
      "asset": { "hash": "f45b1131...", "id": 2, "name": "TOK2", "decimals": 4 },
      "isLiquidity": false,
      "toAddress": "00000000...",
      "tokenSpec": "asset:f45b1131..."
    },
    "hash": "87a8f53490f32a2...",
    "signedCommon": {
      "fee": { "E8": 9992, "str": "0.00009992" },
      "nonceId": 1,
      "originAddress": "3661579d...",
      "originId": 12344,
      "pinHeight": 0
    }
  }
}
```

| Field | Type | Description |
|-------|------|-------------|
| `historyId` | uint64_t | History entry ID |
| `transaction.data.amount` | `FundsDecimal` | Transfer amount |
| `transaction.data.asset` | `AssetBasic` | Asset details |
| `transaction.data.isLiquidity` | boolean | True if this is a liquidity pool share transfer |
| `transaction.data.toAddress` | string | Recipient address |
| `transaction.data.tokenSpec` | string | Token specification string |
| `transaction.hash` | string | Transaction hash |
| `transaction.signedCommon` | object | Signing metadata |

## New Order

A limit order placed on the DEX. The order may have been immediately matched if a counter-order existed in the same block. This is a signed transaction.

```json
{
  "historyId": 15,
  "transaction": {
    "data": {
      "amount": { "str": "0.00010000", "u64": 10000, "decimals": 8 },
      "baseAsset": { "hash": "0e4825ef...", "id": 7, "name": "TOK2", "decimals": 4 },
      "buy": true,
      "limit": {
        "precExponent10": 4,
        "exponent2": -2,
        "mantissa": 40000,
        "hex": "9c404d",
        "doubleAdjusted": 1,
        "doubleRaw": 10000
      }
    },
    "hash": "ed0ac4049643f1b...",
    "processed": {
      "remaining": { "str": "0", "u64": 0, "decimals": 4 }
    },
    "signedCommon": {
      "fee": { "E8": 1, "str": "0.00000001" },
      "nonceId": 111,
      "originAddress": "2de77d5e...",
      "originId": 12344,
      "pinHeight": 0
    }
  }
}
```

| Field | Type | Description |
|-------|------|-------------|
| `historyId` | uint64_t | History entry ID |
| `transaction.data.amount` | `FundsDecimal` | Order amount |
| `transaction.data.baseAsset` | `AssetBasic` | Base asset (traded against WART) |
| `transaction.data.buy` | boolean | True if buy order (pays WART), false if sell |
| `transaction.data.limit` | `PriceDetail` | Limit price |
| `transaction.hash` | string | Transaction hash |
| `transaction.processed.remaining` | `FundsDecimal` | Remaining amount not matched |
| `transaction.signedCommon` | object | Signing metadata |

## Match

Batch-matched orders from a single block. Order matching is based on CoinFuMasterShifu's paper on [Fair Batch Matching](https://github.com/CoinFuMasterShifu/FairBatchMatching/blob/main/FairBatchMatching.pdf). This algorithm batch-matches discrete with continuous liquidity in a fair way such that no intrinsic ordering exists among the matched orders, which means that the algorithm is MEV-proof — in particular, no sandwich attacks are possible.

The returned JSON object contains the pool before and after the matching, and all swaps executed. Each swap includes the history id which can be used for further lookup on the referenced order. This is an unsigned transaction.

```json
{
  "historyId": 38,
  "transaction": {
    "data": {
      "baseAsset": { "hash": "0e4825ef...", "id": 7, "name": "TOK2", "decimals": 4 },
      "poolBefore": {
        "base": { "str": "0", "u64": 0, "decimals": 4 },
        "quote": { "str": "0", "E8": 0 }
      },
      "poolAfter": {
        "base": { "str": "0", "u64": 0, "decimals": 4 },
        "quote": { "str": "0", "E8": 0 }
      },
      "buySwaps": [
        {
          "swapped": {
            "base": { "str": "10.0000", "u64": 100000, "decimals": 4 },
            "quote": { "str": "1.00000000", "E8": 100000000 }
          },
          "historyId": 26
        }
      ],
      "sellSwaps": [
        {
          "swapped": {
            "base": { "str": "20.0010", "u64": 200010, "decimals": 4 },
            "quote": { "str": "2.00010000", "E8": 200010000 }
          },
          "historyId": 25
        }
      ]
    },
    "hash": "2ecefa2fb405cd4..."
  }
}
```

| Field | Type | Description |
|-------|------|-------------|
| `historyId` | uint64_t | History entry ID |
| `transaction.data.baseAsset` | `AssetBasic` | Base asset |
| `transaction.data.poolBefore` | `BaseQuote` | Liquidity pool state before matching |
| `transaction.data.poolAfter` | `BaseQuote` | Liquidity pool state after matching |
| `transaction.data.buySwaps[]` | array | Buy-side swap entries |
| `transaction.data.buySwaps[].swapped` | `BaseQuote` | Swapped amounts |
| `transaction.data.buySwaps[].historyId` | uint64_t | Referenced order history ID |
| `transaction.data.sellSwaps[]` | array | Sell-side swap entries (same fields as `buySwaps`) |
| `transaction.hash` | string | Transaction hash |

## Liquidity Deposit

Deposit of base asset and WART into a liquidity pool, receiving LP shares. This is a signed transaction.

```json
{
  "historyId": 43,
  "transaction": {
    "data": {
      "baseAsset": { "hash": "0e4825ef...", "id": 7, "name": "TOK2", "decimals": 4 },
      "deposited": {
        "base": { "str": "100.0000", "u64": 1000000, "decimals": 4 },
        "quote": { "str": "1.00000000", "E8": 100000000 }
      }
    },
    "hash": "1db23925133e7d3...",
    "processed": {
      "sharesReceived": { "str": "1000.0000", "u64": 10000000, "decimals": 4 }
    },
    "signedCommon": {
      "fee": { "E8": 1, "str": "0.00000001" },
      "nonceId": 111111,
      "originAddress": "2de77d5e...",
      "originId": 12344,
      "pinHeight": 0
    }
  }
}
```

| Field | Type | Description |
|-------|------|-------------|
| `historyId` | uint64_t | History entry ID |
| `transaction.data.baseAsset` | `AssetBasic` | Base asset |
| `transaction.data.deposited` | `BaseQuote` | Base and WART amounts deposited |
| `transaction.hash` | string | Transaction hash |
| `transaction.processed.sharesReceived` | `FundsDecimal` | LP shares received |
| `transaction.signedCommon` | object | Signing metadata |

## Liquidity Withdrawal

Withdrawal of base asset and WART from a liquidity pool by redeeming LP shares. This is a signed transaction.

```json
{
  "historyId": 42,
  "transaction": {
    "data": {
      "baseAsset": { "hash": "0e4825ef...", "id": 7, "name": "TOK2", "decimals": 4 },
      "sharesRedeemed": { "str": "100.0000", "u64": 1000000, "decimals": 4 }
    },
    "hash": "47c50348397eed9...",
    "processed": {
      "received": {
        "base": { "str": "100.0000", "u64": 1000000, "decimals": 4 },
        "quote": { "str": "1.00000000", "E8": 100000000 }
      }
    },
    "signedCommon": {
      "fee": { "E8": 9992, "str": "0.00009992" },
      "nonceId": 3,
      "originAddress": "3661579d...",
      "originId": 12344,
      "pinHeight": 0
    }
  }
}
```

| Field | Type | Description |
|-------|------|-------------|
| `historyId` | uint64_t | History entry ID |
| `transaction.data.baseAsset` | `AssetBasic` | Base asset |
| `transaction.data.sharesRedeemed` | `FundsDecimal` | LP shares redeemed |
| `transaction.hash` | string | Transaction hash |
| `transaction.processed.received` | `BaseQuote` | Base and WART amounts received |
| `transaction.signedCommon` | object | Signing metadata |

## Asset Creation

Creation of a new token. This is a signed transaction.

```json
{
  "historyId": 10,
  "transaction": {
    "data": {
      "name": "TOK2",
      "supply": { "str": "100000000.0000", "u64": 1000000000000, "decimals": 4 }
    },
    "hash": "0e4825efffa2946...",
    "processed": {
      "assetId": 7
    },
    "signedCommon": {
      "fee": { "E8": 9992, "str": "0.00009992" },
      "nonceId": 13,
      "originAddress": "3661579d...",
      "originId": 12344,
      "pinHeight": 0
    }
  }
}
```

| Field | Type | Description |
|-------|------|-------------|
| `historyId` | uint64_t | History entry ID |
| `transaction.data.name` | string | Token name |
| `transaction.data.supply` | `FundsDecimal` | Total token supply |
| `transaction.hash` | string | Transaction hash |
| `transaction.processed.assetId` | uint64_t | Assigned asset ID |
| `transaction.signedCommon` | object | Signing metadata |

## Cancelation

A transaction cancelation. Cancel transactions block other transactions by referencing them — no transaction with the referenced transaction ID can be mined in the same or a future block. This is a signed transaction.

```json
{
  "historyId": 47,
  "transaction": {
    "data": {
      "cancelTxid": {
        "accountId": 2,
        "nonceId": 3,
        "pinHeight": 10
      }
    },
    "hash": "a3aadbaa...",
    "processed": {
      "canceledTxHash": "..."
    },
    "signedCommon": {
      "fee": { "E8": 9992, "str": "0.00009992" },
      "nonceId": 5,
      "originAddress": "3661579d...",
      "originId": 12344,
      "pinHeight": 0
    }
  }
}
```

| Field | Type | Description |
|-------|------|-------------|
| `historyId` | uint64_t | History entry ID |
| `transaction.data.cancelTxid` | object | Reference to the cancelled transaction |
| `transaction.data.cancelTxid.accountId` | uint64_t | Account ID of cancelled transaction |
| `transaction.data.cancelTxid.nonceId` | uint32_t | Nonce ID of cancelled transaction |
| `transaction.data.cancelTxid.pinHeight` | uint32_t | Pin height of cancelled transaction |
| `transaction.hash` | string | Transaction hash |
| `transaction.processed.canceledTxHash` | string | Hash of the cancelled transaction |
| `transaction.signedCommon` | object | Signing metadata |
