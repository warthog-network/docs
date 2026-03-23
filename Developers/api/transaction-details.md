# Transaction Details

All block body arrays share the same entry structures described below. See [GET /transaction/latest](#get-transactionlatest) for context on where these fields appear.

## Rewards

Block rewards paid to miners.

```json
{
 "txHash": "e849a0fe...",
 "historyId": 46,
 "toAddress": "00000000...de47c9b2",
 "amount": { "E8": 300019985, "str": "3.00019985" }
}
```

| Field | Type | Description |
|-------|------|-------------|
| `txHash` | string | Transaction hash (hex) |
| `historyId` | integer | History entry ID (null if pending) |
| `toAddress` | string | Recipient address |
| `amount.E8` | integer | Amount in raw E8 units |
| `amount.str` | string | Amount as decimal string |

## Wart Transfers

WART coin transfers between accounts.

```json
{
 "txHash": "897b3cc7...",
 "historyId": 42,
 "fromAddress": "3661579d...",
 "fee": { "E8": 9992, "str": "0.00009992" },
 "nonceId": 0,
 "pinHeight": 0,
 "toAddress": "00000000...de47c9b2",
 "amount": { "E8": 100000000, "str": "1.00000000" }
}
```

| Field | Type | Description |
|-------|------|-------------|
| `txHash` | string | Transaction hash |
| `historyId` | integer | History entry ID |
| `fromAddress` | string | Sender address |
| `fee.E8` | integer | Fee in raw E8 units |
| `fee.str` | string | Fee as decimal string |
| `nonceId` | integer | Nonce ID |
| `pinHeight` | integer | Pin block height |
| `toAddress` | string | Recipient address |
| `amount.E8` | integer | Amount in raw E8 units |
| `amount.str` | string | Amount as decimal string |

## Token Transfers

Token transfers between accounts.

```json
{
 "txHash": "edb03806...",
 "historyId": 48,
 "fromAddress": "2de77d5e...",
 "fee": { "E8": 1, "str": "0.00000001" },
 "nonceId": 100,
 "pinHeight": 0,
 "toAddress": "00000000...de47c9b2",
 "amount": { "str": "100.0000", "u64": 1000000, "decimals": 4 },
 "asset": { "id": 2, "name": "TOK2", "decimals": 4, "hash": "f45b1131..." },
 "isLiquidity": false,
 "tokenSpec": "asset:f45b1131..."
}
```

| Field | Type | Description |
|-------|------|-------------|
| `txHash` | string | Transaction hash |
| `historyId` | integer | History entry ID |
| `fromAddress` | string | Sender address |
| `fee.E8` | integer | Fee in raw E8 units |
| `fee.str` | string | Fee as decimal string |
| `nonceId` | integer | Nonce ID |
| `pinHeight` | integer | Pin block height |
| `toAddress` | string | Recipient address |
| `amount.str` | string | Amount as decimal string |
| `amount.u64` | integer | Amount in smallest unit |
| `amount.decimals` | integer | Token decimals |
| `asset.id` | integer | Asset ID |
| `asset.name` | string | Asset name |
| `asset.decimals` | integer | Asset decimals |
| `asset.hash` | string | Asset hash |
| `isLiquidity` | boolean | Whether transfer is liquidity-related |
| `tokenSpec` | string | Token specification string |

## New Orders

New limit orders placed on the DEX.

```json
{
 "txHash": "c6fbade2...",
 "historyId": 37,
 "address": "2de77d5e...",
 "fee": { "E8": 1, "str": "0.00000001" },
 "nonceId": 99,
 "pinHeight": 0,
 "baseAsset": { "id": 7, "name": "TOK2", "decimals": 4, "hash": "0e4825ef..." },
 "amount": { "str": "100.0000", "u64": 1000000, "decimals": 4 },
 "limit": { "precExponent10": 4, "exponent2": -6, "mantissa": 64000, "hex": "fa0049", "doubleAdjusted": 0.1, "doubleRaw": 1000.0 },
 "buy": false
}
```

| Field | Type | Description |
|-------|------|-------------|
| `txHash` | string | Transaction hash |
| `historyId` | integer | History entry ID |
| `address` | string | Order creator address |
| `fee.E8` | integer | Fee in raw E8 units |
| `fee.str` | string | Fee as decimal string |
| `nonceId` | integer | Nonce ID |
| `pinHeight` | integer | Pin block height |
| `baseAsset.id` | integer | Base asset ID |
| `baseAsset.name` | string | Base asset name |
| `baseAsset.decimals` | integer | Base asset decimals |
| `baseAsset.hash` | string | Base asset hash |
| `amount.str` | string | Order amount as decimal string |
| `amount.u64` | integer | Order amount in smallest unit |
| `amount.decimals` | integer | Amount decimals |
| `limit.precExponent10` | integer | Base-10 precision exponent |
| `limit.exponent2` | integer | Power-of-2 exponent |
| `limit.mantissa` | integer | 16-bit mantissa |
| `limit.hex` | string | Raw bytes (hex) |
| `limit.doubleAdjusted` | number | Price as double (adjusted) |
| `limit.doubleRaw` | number | Price as double (raw) |
| `buy` | boolean | True if buy order, false if sell |

## Matches

DEX trade matches (swaps between liquidity providers).

```json
{
 "txHash": "2ecefa2f...",
 "historyId": 38,
 "baseAsset": { "id": 7, "name": "TOK2", "decimals": 4, "hash": "0e4825ef..." },
 "poolBefore": {
  "base": { "str": "0", "u64": 0, "decimals": 4 },
  "quote": { "str": "0", "E8": 0 }
 },
 "poolAfter": {
  "base": { "str": "0", "u64": 0, "decimals": 4 },
  "quote": { "str": "0", "E8": 0 }
 },
 "buySwaps": [
  { "base": { "str": "10.0000", "u64": 100000, "decimals": 4 }, "quote": { "str": "1.00000000", "E8": 100000000 }, "historyId": 26 }
 ],
 "sellSwaps": [
  { "base": { "str": "20.0010", "u64": 200010, "decimals": 4 }, "quote": { "str": "2.00010000", "E8": 200010000 }, "historyId": 25 }
 ]
}
```

| Field | Type | Description |
|-------|------|-------------|
| `txHash` | string | Transaction hash |
| `historyId` | integer | History entry ID |
| `baseAsset.id` | integer | Base asset ID |
| `baseAsset.name` | string | Base asset name |
| `baseAsset.decimals` | integer | Base asset decimals |
| `baseAsset.hash` | string | Base asset hash |
| `poolBefore.base.str` | string | Base pool before (decimal) |
| `poolBefore.base.u64` | integer | Base pool before (smallest unit) |
| `poolBefore.base.decimals` | integer | Base pool decimals |
| `poolBefore.quote.str` | string | Quote pool before (decimal) |
| `poolBefore.quote.E8` | integer | Quote pool before (E8) |
| `poolAfter.*` | | Same structure as `poolBefore` |
| `buySwaps[]` | array | Buy-side swap entries |
| `buySwaps[].base.str` | string | Swap base amount (decimal) |
| `buySwaps[].base.u64` | integer | Swap base amount (smallest unit) |
| `buySwaps[].base.decimals` | integer | Swap base decimals |
| `buySwaps[].quote.str` | string | Swap quote amount (decimal) |
| `buySwaps[].quote.E8` | integer | Swap quote amount (E8) |
| `buySwaps[].historyId` | integer | Referenced history ID |
| `sellSwaps[]` | array | Sell-side swap entries (same fields as `buySwaps`) |

## Liquidity Deposits

Liquidity provider deposits into a trading pool.

```json
{
 "txHash": "576da5fd...",
 "historyId": 11,
 "address": "3661579d...",
 "fee": { "E8": 9992, "str": "0.00009992" },
 "nonceId": 0,
 "pinHeight": 0,
 "baseAsset": { "id": 7, "name": "TOK2", "decimals": 4, "hash": "0e4825ef..." },
 "baseDeposited": { "str": "100.0000", "u64": 1000000, "decimals": 4 },
 "quoteDeposited": { "str": "1.00000000", "E8": 100000000 },
 "sharesReceived": { "str": "10.0000", "u64": 100000, "decimals": 4 }
}
```

| Field | Type | Description |
|-------|------|-------------|
| `txHash` | string | Transaction hash |
| `historyId` | integer | History entry ID |
| `address` | string | Depositor address |
| `fee.E8` | integer | Fee in raw E8 units |
| `fee.str` | string | Fee as decimal string |
| `nonceId` | integer | Nonce ID |
| `pinHeight` | integer | Pin block height |
| `baseAsset.*` | object | Base asset info (see New Orders) |
| `baseDeposited.str` | string | Base amount deposited (decimal) |
| `baseDeposited.u64` | integer | Base amount deposited (smallest unit) |
| `baseDeposited.decimals` | integer | Base decimals |
| `quoteDeposited.str` | string | Quote amount deposited (decimal) |
| `quoteDeposited.E8` | integer | Quote amount deposited (E8) |
| `sharesReceived` | object | LP shares received (null if none, same fields as `baseDeposited`) |

## Liquidity Withdrawals

Liquidity provider withdrawals from a trading pool.

```json
{
 "txHash": "576da5fd...",
 "historyId": 11,
 "address": "3661579d...",
 "fee": { "E8": 9992, "str": "0.00009992" },
 "nonceId": 0,
 "pinHeight": 0,
 "baseAsset": { "id": 7, "name": "TOK2", "decimals": 4, "hash": "0e4825ef..." },
 "sharesRedeemed": { "str": "10.0000", "u64": 100000, "decimals": 4 },
 "baseReceived": { "str": "100.0000", "u64": 1000000, "decimals": 4 },
 "quoteReceived": { "str": "1.00000000", "E8": 100000000 }
}
```

| Field | Type | Description |
|-------|------|-------------|
| `txHash` | string | Transaction hash |
| `historyId` | integer | History entry ID |
| `address` | string | Withdrawer address |
| `fee.E8` | integer | Fee in raw E8 units |
| `fee.str` | string | Fee as decimal string |
| `nonceId` | integer | Nonce ID |
| `pinHeight` | integer | Pin block height |
| `baseAsset.*` | object | Base asset info (see New Orders) |
| `sharesRedeemed.str` | string | Shares redeemed (decimal) |
| `sharesRedeemed.u64` | integer | Shares redeemed (smallest unit) |
| `sharesRedeemed.decimals` | integer | Shares decimals |
| `baseReceived` | object | Base asset received (null if none, same fields as `baseDeposited`) |
| `quoteReceived` | object | Quote asset received (null if none, same fields as `quoteDeposited`) |

## Asset Creations

New token/asset creation transactions.

```json
{
 "txHash": "0e4825ef...",
 "historyId": 10,
 "creatorAddress": "3661579d...",
 "fee": { "E8": 9992, "str": "0.00009992" },
 "nonceId": 13,
 "pinHeight": 0,
 "supply": "100000000.0000",
 "name": "TOK2",
 "assetId": 7
}
```

| Field | Type | Description |
|-------|------|-------------|
| `txHash` | string | Transaction hash |
| `historyId` | integer | History entry ID |
| `creatorAddress` | string | Creator address |
| `fee.E8` | integer | Fee in raw E8 units |
| `fee.str` | string | Fee as decimal string |
| `nonceId` | integer | Nonce ID |
| `pinHeight` | integer | Pin block height |
| `supply` | string | Total token supply (decimal) |
| `name` | string | Token name |
| `assetId` | integer | Assigned asset ID (null if pending) |

## Cancelations

Transaction cancelations (revert pending transactions).

```json
{
 "txHash": "a3aadbaa...",
 "historyId": 47,
 "address": "3661579d...",
 "fee": { "E8": 9992, "str": "0.00009992" },
 "nonceId": 5,
 "pinHeight": 0
}
```

| Field | Type | Description |
|-------|------|-------------|
| `txHash` | string | Transaction hash |
| `historyId` | integer | History entry ID |
| `address` | string | Canceling address |
| `fee.E8` | integer | Fee in raw E8 units |
| `fee.str` | string | Fee as decimal string |
| `nonceId` | integer | Nonce ID of cancelled tx |
| `pinHeight` | integer | Pin block height |

## Order Cancelations

Order cancelations (cancel limit orders on the DEX).

```json
{
 "txHash": "a3aadbaa...",
 "historyId": 47,
 "cancelTxid": { "accountId": 2, "nonceId": 3, "pinHeight": 10 },
 "buy": true,
 "baseAsset": { "id": 7, "name": "TOK2", "decimals": 4, "hash": "0e4825ef..." },
 "remaining": { "E8": 100000000, "str": "1.00000000" }
}
```

| Field | Type | Description |
|-------|------|-------------|
| `txHash` | string | Transaction hash |
| `historyId` | integer | History entry ID |
| `cancelTxid.accountId` | integer | Account ID of cancelled order |
| `cancelTxid.nonceId` | integer | Nonce ID of cancelled order |
| `cancelTxid.pinHeight` | integer | Pin height of cancelled order |
| `buy` | boolean | True if buy order, false if sell |
| `baseAsset.*` | object | Base asset info (see New Orders) |
| `remaining.E8` | integer | Remaining amount in E8 units |
| `remaining.str` | string | Remaining amount as decimal string |
