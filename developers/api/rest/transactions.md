# Transactions
## Structure
Warthog nodes return two types of transactions:
### Signed Transactions
Such transactions are signed. Their API structure is as follows:

```json
  "transaction": {
    "data": { ... },
    "hash": "...",
    "signedCommon": { ... }
    "processed": { ... }
  }
```
Currently Warthog supports the following signed transactions:

1. **WartTransfer** Transfer WART to another account.
2. **TokenTransfers** Transfers a non-WART token to another account.
3. **AssetCreation** Creates a new asset.
4. **Cancelations** Cancels a transaction if mined first and also cancels a pending order from the order book.
5. **LiquidityDeposits** Deposits liquidity into a liquidity pool.
6. **LiquidityWithdrawals** Withdraws liquidity from a liquidity pool.
7. **NewOrders** Creates a new order for a specific WART trading pair (Warthog only supports pairs with WART as quote currency).

### Induced Transactions
These are transactions that are induced by the logic of how Warthog processes a block.
Currently we have the following induced transactions

8. **Reward** This transaction represents the block reward.
9. **Match** This transaction represents a match of three DeFi liquidity sources: buy swap orders, sell swap orders and the liquidity pool. The swaps are executed using Warthog's custom [matching engine](/unique-features/sandwich-proof-defi/matching-engine.md)

    Sinc induced transactions were not signed by any user but are rather a side effect of block processing, compared to signed transactions we do not have the `signedCommon` fields but apart from that, the structure is similar: 
```json
  "transaction": {
    "data": { ... },
    "hash": "...",
    "processed": { ... }
  }
```

## Transaction properties
We now explain the four properties `data`, `hash`, `signedCommon` and `processed`. 
### Property `signedCommon`
The `signedCommon` property only exists in signed transactions. It always has the structure
```json
"signedCommon": {
  "fee": {                          // fee used for the transaction
    "E8": 9992,                     // integer representation of the fee (multiplied by 10^8)
    "str": "0.00009992"             // string representation  of the fee
  },
  "nonceId": 1,                     // nonce id used to sign the transaction
  "originAddress": "3661...e8f5a",  // signer/origin address
  "originId": 12344,                // signer/origin account id
  "pinHeight": 0                    // pin height used to sign the transaction
}
```
### Property `hash`
Every transaction has a transaction hash. For signed transactions this is the hash that is used to generate the signature. For induced transactions this hash is generated when the block is included in the chain.
### Properties `data` and `processed`

`data` contains the user-supplied transaction parameters (what is signed). `processed` contains the chain-computed result — present only for transactions that go through block processing (NewOrder, LiquidityDeposit, LiquidityWithdrawal, AssetCreation, Cancelation). Reward and Match are unsigned (no `signedCommon`) and have no `processed`.

#### 1. Reward (`data` only — unsigned, no `processed`)

Block reward paid to the miner who produced the block. Unsigned — it is a side effect of block production, not submitted by a user. No `signedCommon`, no `processed`.

```json
"data": {
  "amount": {                  // block reward paid to the miner
    "E8": 300000000,           // integer representation (1 WART = 10^8 E8)
    "str": "3.00000000"        // human-readable decimal string
  },
  "toAddress": "3661...e8f5a"  // miner receiving the reward
}
```

#### 2. WartTransfer (`data` only — signed, no `processed`)

WART coin transfer from one account to another. Signed — the sender signs the transaction. No `processed` since a simple transfer has no chain-computed result beyond the balance change.

```json
"data": {
  "amount": {                  // WART amount transferred
    "E8": 100000000,           // integer representation (1 WART = 10^8 E8)
    "str": "1.00000000"        // human-readable decimal string
  },
  "toAddress": "0000000000000000000000000000000000000000000000000000000000000000" // recipient address
}
```

#### 3. TokenTransfer (`data` only — signed, no `processed`)

Transfer of a non-WART token between accounts. `isLiquidity` is `true` if the transferred token is a liquidity pool share. Signed. No `processed` — the balance changes are the result.

```json
"data": {
  "amount": {                       // token amount transferred
    "decimals": 4,                  // token decimal places
    "str": "99.9912",               // human-readable decimal string
    "u64": 999912                   // integer representation in smallest unit
  },
  "asset": {                        // asset being transferred
    "decimals": 4,                  // asset decimal places
    "hash": "f45b...5a6b",          // asset hash
    "id": 2,                        // assigned asset ID
    "name": "TOK2"                  // asset name
  },
  "isLiquidity": false,             // true if this is a liquidity pool share transfer
  "toAddress": "0000...0000",       // recipient address
  "tokenSpec": "asset:f45b1...5a6b" // token specification string
}
```

#### 4. NewOrder (`data` + `processed`)

A limit order on the DEX for a trading pair where WART is always the quote currency. Signed. The `processed.remaining` field shows how much of the order was not matched in the same block — if fully matched, `remaining` is zero. A non-zero `remaining` means the order (or remainder) stays in the order book.

```json
"data": {                      // what the user signed (mirrored in processed)
  "amount": {                  // order amount
    "decimals": 4,
    "str": "0.00010000",
    "u64": 10000
  },
  "baseAsset": {               // asset being traded (WART is always the quote)
    "decimals": 4,
    "hash": "0e4825ef123456789abcdef0123456789abcdef0123456789abcdef0123",
    "id": 7,
    "name": "TOK2"
  },
  "buy": true,                 // true = buy order (pays WART), false = sell order
  "limit": {                   // limit price (exact price for this order)
    "doubleAdjusted": 1.0,     // price as double (adjusted)
    "doubleRaw": 10000.0,      // price as double (raw)
    "exponent2": -2,           // power-of-2 exponent
    "hex": "9c404d",           // raw price bytes (hex)
    "mantissa": 40000,         // 16-bit mantissa
    "precExponent10": 4        // base-10 precision exponent
  }
},
```

Processed:
```json
"processed": {
  "remaining": {                // amount of the order that was NOT matched in this block
    "decimals": 4,
    "str": "0.00000000",
    "u64": 0
  }
}
```

#### 5. Match (`data` only — unsigned, no `processed`)

Batch-matched orders from a single block. Unsigned — this is a side effect of block processing. The `poolBefore`/`poolAfter` fields show the liquidity pool state before and after this match. Each entry in `buySwaps` and `sellSwaps` references the matched order via `historyId`. Matching uses Fair Batch Matching (see the [matching engine](/unique-features/sandwich-proof-defi/matching-engine.md)) — no intrinsic ordering among matched orders, making the algorithm MEV-proof.

```json
"data": {
  "baseAsset": {                // asset being traded against WART
    "decimals": 4,
    "hash": "0e4825ef123456789abcdef0123456789abcdef0123456789abcdef0123",
    "id": 7,
    "name": "TOK2"
  },
  "poolBefore": {               // liquidity pool state before this match
    "base": { "decimals": 4, "str": "1000.0000", "u64": 10000000 },
    "quote": { "E8": 100000000, "str": "1.00000000" }
  },
  "poolAfter": {                // liquidity pool state after this match
    "base": { "decimals": 4, "str": "1010.0000", "u64": 10100000 },
    "quote": { "E8": 110000000, "str": "1.10000000" }
  },
  "buySwaps": [                 // swaps resulting from buy orders being matched
    {
      "swapped": {              // amounts swapped in this individual trade
        "base": { "decimals": 4, "str": "10.0000", "u64": 100000 },
        "quote": { "E8": 100000000, "str": "1.00000000" }
      },
      "historyId": 26           // history ID of the matched buy order
    }
  ],
  "sellSwaps": [                // swaps resulting from sell orders being matched
    {
      "swapped": {
        "base": { "decimals": 4, "str": "20.0010", "u64": 200010 },
        "quote": { "E8": 200010000, "str": "2.00010000" }
      },
      "historyId": 25           // history ID of the matched sell order
    }
  ]
}
```

#### 6. LiquidityDeposit (`data` + `processed`)

Deposit of a base asset and WART into a liquidity pool, receiving LP shares in return. Signed. `processed.sharesReceived` is the amount of liquidity pool shares minted to the depositor.

```json
"data": {
  "baseAsset": {                // asset being deposited alongside WART
    "decimals": 4,
    "hash": "0e4825ef123456789abcdef0123456789abcdef0123456789abcdef0123",
    "id": 7,
    "name": "TOK2"
  },
  "deposited": {                // amounts deposited into the liquidity pool
    "base": { "decimals": 4, "str": "100.0000", "u64": 1000000 },
    "quote": { "E8": 100000000, "str": "1.00000000" }
  }
},
```

Processed:
```json
"processed": {
  "sharesReceived": {           // liquidity pool shares minted to the depositor
    "decimals": 4,
    "str": "1000.0000",
    "u64": 10000000
  }
}
```

#### 7. LiquidityWithdrawal (`data` + `processed`)

Redemption of liquidity pool shares to withdraw the underlying base asset and WART. Signed. `processed.received` shows the actual amounts withdrawn, which may differ slightly from the proportional share due to rounding or pool state changes.

```json
"data": {
  "baseAsset": {                // asset being withdrawn alongside WART
    "decimals": 4,
    "hash": "0e4825ef123456789abcdef0123456789abcdef0123456789abcdef0123",
    "id": 7,
    "name": "TOK2"
  },
  "sharesRedeemed": {           // liquidity pool shares being burned
    "decimals": 4,
    "str": "100.0000",
    "u64": 1000000
  }
},
```

Processed:
```json
"processed": {
  "received": {                 // amounts withdrawn from the liquidity pool
    "base": { "decimals": 4, "str": "100.0000", "u64": 1000000 },
    "quote": { "E8": 100000000, "str": "1.00000000" }
  }
}
```

#### 8. AssetCreation (`data` + `processed`)

Creation of a new token with a given name and total supply. Signed. `processed.assetId` is the ID assigned to the newly created asset.

```json
"data": {
  "name": "TOK2",               // token name
  "supply": {                   // total token supply
    "decimals": 4,
    "str": "100000000.0000",
    "u64": 1000000000000
  }
},
```

Processed:
```json
"processed": {
  "assetId": 7                  // ID assigned to the newly created asset
}
```

#### 9. Cancelation (`data` + `processed`)

Cancellation by referencing a target transaction via its `TransactionId` (`accountId`, `nonceId`, `pinHeight`). Signed. Once mined, no transaction with the referenced ID can be included in the same or any future block — effectively blocking it from ever being mined. `processed.canceledTxHash` confirms the hash of the cancelled transaction.

==- data
```json
"data": {
  "cancelTxid": {                 // reference to the transaction being cancelled
    "accountId": 2,               // account ID of the transaction to cancel
    "nonceId": 3,                 // nonce ID of the transaction to cancel
    "pinHeight": 10               // pin height of the transaction to cancel
  }
},
```

==- Processed
```json
"processed": {
  "canceledTxHash": "ed0a...2f3a" // hash of the cancelled transaction
}
```
===
