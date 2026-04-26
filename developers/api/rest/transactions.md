---
order: 1
---

# Transactions

Warthog supports the following transactions, some of which are signed and can have a `processed` property:

| Type | Explanation |  signed | `processed`
|---|---|---|
| `wartTransfer` | Transfer WART to another account | X|
| `tokenTransfer` | Transfer a non-WART token to another account | X |
| `assetCreation` | Create a new asset | X | X
| `cancelation` | Cancel a transaction / pending order |  X | X
| `liquidityDeposit` | Deposit liquidity into a liquidity pool | X | X
| `liquidityWithdrawal` | Withdraw liquidity from a liquidity pool | X | X
| `limitSwap` | Create a limit order on the DEX | X | X
| `reward` | Block reward paid to miner |  |
| `match` | Batch-matched orders from a block |  |

## Type Specification
The API returns transactions either in a block or individually.
### Within Block
In a block, transactions are grouped by their type such that the type of each transaction is clear:
```json
{
 "body":{
  "reward": ...                  // null or reward transaction
  "wartTransfer": [...],         // contains wartTransfer transactions
  "tokenTransfer": [...],        // contains tokenTransfer transactions
  "limitSwap": [...],            // contains limitSwap transactions
  "match": [...],                // contains match transactions
  "liquidityDeposit": [...],     // contains liquidityDeposit transactions
  "liquidityWithdrawal": [...],  // contains liquidityWithdrawal transactions
  "assetCreation": [...],        // contains assetCreation transactions
  "cancelation": [...]           // contains cancelation transactions
 }
}
```
### Individually
When a single transaction is returned, its type is annotated by a `type` property:
```json
{
 "type": "<type>",     // String like "wartTransfer", "tokenTransfer", etc.
 "transaction": {...}, // Actual transaction object
 ...
}
```



## Signed and Implicit Transactions
### Signed Transactions
Signed transactions are signed by some address. Their API structure is

```json
{
  "data": { ... },
  "hash": "...",
  "signedCommon": {
   "fee": {                          // fee used for the transaction
     "E8": 9992,                     // integer representation of the fee (multiplied by 10^8)
     "str": "0.00009992"             // string representation  of the fee
   },
   "nonceId": 1,                     // nonce id used to sign the transaction
   "originAddress": "3661...e8f5a",  // signer/origin address
   "originId": 12344,                // signer/origin account id
   "pinHeight": 0                    // pin height used to sign the transaction
  },
  "processed": { ... }               // Not always present, see below
}
```
The `hash` property contains the transaction hash that is used to generate the signature. The structure of the `data` and `processed` properties depends on the actual transaction type. The `processed` property does not exist for all transaction types and provides information about the outcome or current state of the transaction **after** it was mined.

The following 7 transaction types are signed:

#### 1. WartTransfer (no `processed` state)

WART coin transfer from one account to another.

```json
"data": {
  "amount": {                  // WART amount transferred
    "E8": 100000000,           // integer representation (1 WART = 10^8 E8)
    "str": "1.00000000"        // human-readable decimal string
  },
  "toAddress": "0000000000000000000000000000000000000000000000000000000000000000" // recipient address
}
```
If the transaction was mined, the transfer was completed, there is no `processed` state.

#### 2. TokenTransfer (no `processed` state)

Transfer of a non-WART token between accounts. The `isLiquidity` property reveals if the transferred token is the specified asset's pool liquidity or the asset itself..

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
If the transaction was mined, the transfer was completed, there is no `processed` state.

#### 3. `limitSwap` (has `processed` state)

A limit order on the DEX for a trading pair where WART is always the quote currency.
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

After the transaction was mined, the `processed` fields contains information about the currently remaining amount of the order. If the order was already completely filled or canceled, the returned value is 0:
```json
"processed": {
  "remaining": {                // remaining amount of the order
    "decimals": 4,
    "str": "0.00000000",
    "u64": 0
  }
}
```

#### 4. LiquidityDeposit (has `processed` state)

Deposit of a base asset and WART into a liquidity pool, receiving LP shares in return.

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

After the transaction was mined, the `processed` specifies the amount of liquidity pool shares minted to the depositor.
Processed:
```json
"processed": {
  "sharesReceived": {           // liquidity pool shares minted to the depositor
    "decimals": 8,
    "str": "0.1",
    "u64": 10000000
  }
}
```

#### 5. LiquidityWithdrawal (has `processed` state)

Redemption of liquidity pool shares to withdraw the underlying liquidity in base asset and WART.
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

After the transaction was mined, the `processed` property provides information about the received amount of base asset and WART (`quote`):
```json
"processed": {
  "received": {                 // amounts withdrawn from the liquidity pool
    "base": { "decimals": 4, "str": "100.0000", "u64": 1000000 },
    "quote": { "E8": 100000000, "str": "1.00000000" }
  }
}
```

#### 6. AssetCreation (has `processed` state)

Creation of a new token with a given name and total supply.

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

After the transaction was mined, the `processed` property provides the assigned asset ID. This ID is permanent but note that in case the block containing the asset generation was rolled back, a different ID could have been assigned to the asset. Therefore it is safer to always address assets by hash. The asset hash is the hash of the transaction creating the asset which can be extracted from the `hash` property if the transaction.
```json
"processed": {
  "assetId": 7                  // ID assigned to the newly created asset
}
```

#### 7. Cancelation (has `processed` state)

This transaction is used to cancel a pending (not yet mined) transaction and also to cancel an open (not completely filled) order from the order book. The target transaction is defined by its `TransactionId` (`accountId`, `nonceId`, `pinHeight`). Once mined, no transaction with the referenced ID can be included in the same or any future block, effectively blocking it from ever being mined. Also, if an open order can be matched, it will be canceled.

```json
"data": {
  "cancelTxid": {                 // reference to the transaction being cancelled
    "accountId": 2,               // account ID of the transaction to cancel
    "nonceId": 3,                 // nonce ID of the transaction to cancel
    "pinHeight": 10               // pin height of the transaction to cancel
  }
},
```

After the transaction was mined the `processed` states provides the hash of the canceled active order, or `null` if no such active order was matched by the cancelation.
```json
"processed": {
  "canceledTxHash": ... // either null or string (hash of the cancelled order)
}
```

### Implicit Transactions
Implicit transactions represent implicitly generated side effects during block processing. They don't have a `signedCommon` property and for now they also never have a `processed` state:
```json
  "transaction": {
    "data": { ... },
    "hash": "...",
  }
```
The `hash` is automatically generated when the block is included in the chain whereas the structure of the `data` property depends on the actual transaction type. There are two implicit transactions.

#### 1. Reward
Block reward paid to the miner who produced the block.

```json
"data": {
  "amount": {                  // block reward paid to the miner
    "E8": 300000000,           // integer representation (1 WART = 10^8 E8)
    "str": "3.00000000"        // human-readable decimal string
  },
  "toAddress": "3661...e8f5a"  // miner receiving the reward
}
```

#### 2. Match
Match of three DeFi liquidity sources using Warthog's custom [matching engine](/unique-features/sandwich-proof-defi/matching-engine.md): buy swap orders, sell swap orders and the liquidity pool.

The `poolBefore`/`poolAfter` fields show the liquidity pool state before and after this match. Each entry in `buySwaps` and `sellSwaps` references the matched order via `historyId`.
Matching uses Fair Batch Matching (see the [matching engine](/unique-features/sandwich-proof-defi/matching-engine.md)). No intrinsic ordering among matched orders exists, making the algorithm MEV-proof.

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

