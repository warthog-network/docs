---
title: REST
---
# REST API

!!!
This documentation was partially generated with AI augmentation. Report inconsistencies at [github.com/warthog-network/docs/issues](https://github.com/warthog-network/docs/issues).
!!!

API for Warthog node version v0.10.3 "a26330e"

## Configuration

API is accessed via the RPC endpoints of the node. The endpoints socket can be configured via the `--rpc` command line option:

```bash
JSON RPC endpoint options:
  -r, --rpc=IP:PORT          JSON RPC endpoint  (default=`127.0.0.1:3000')
```

For example invoke

```bash
$./wart-node --rpc=0.0.0.0:3000
```

to start the node with RPC listening on all devices on port 3000.

!!!warning Warning
The RPC endpoint should not exposed to the internet, use appropriate firewall settings.
!!!

Below we assume the RPC socket is accessible at `localhost:3000`. On startup the node reports the RPC endpoint setting:

```bash
[info] RPC endpoint is 127.0.0.1:3000.
```

## Overview

| Method | Path | Description |
|--------|------|-------------|
| `POST` | [`/transaction/add`](#post-transactionadd) | Send transactions |
| `GET` | [`/transaction/mempool`](#get-transactionmempool) | Show content of mempool |
| `GET` | [`/transaction/lookup/:txid`](#get-transactionlookuptxid) | Transaction lookup by transaction ID |
| `GET` | [`/transaction/latest`](#get-transactionlatest) | Show latest transactions |
| `GET` | [`/transaction/minfee`](#get-transactionminfee) | Show the minimum fee required by this node |
| `GET` | [`/settings/mempool/minfee/:feeE8`](#get-settingsmempoolminfeefeee8) | Adjust the minimum transaction fee |
| `GET` | [`/chain/head`](#get-chainhead) | Show info on chain head |
| `GET` | [`/chain/grid`](#get-chaingrid) | Show header grid (used for sync) |
| `GET` | [`/chain/block/:id/hash`](#get-chainblockidhash) | Show hash of specific block |
| `GET` | [`/chain/block/:id/header`](#get-chainblockidheader) | Show header of specific block |
| `GET` | [`/chain/block/:id/binary`](#get-chainblockidbinary) | Show binary data of specific block |
| `GET` | [`/chain/block/:id`](#get-chainblockid) | Show header and body of specific block |
| `GET` | [`/chain/mine/:account`](#get-chainmineaccount) | Generate data required for mining |
| `GET` | [`/chain/txcache`](#get-chaintxcache) | Show transaction cache |
| `GET` | [`/chain/hashrate/:window`](#get-chainhashratewindow) | Show current hashrate based on latest N blocks |
| `GET` | [`/chain/signed_snapshot`](#get-chainsigned_snapshot) | Show chain snapshot |
| `POST` | [`/chain/append`](#post-chainappend) | Append mined block |
| `GET` | [`/asset/complete?namePrefix=...&hashPrefix=...`](#get-assetcomplete) | Search assets by name and/or hash prefix |
| `GET` | [`/asset/lookup/:asset`](#get-assetlookupasset) | Asset lookup (by ID or hash) |
| `GET` | [`/dex/market/:market`](#get-dexmarketmarket) | Show market orders and liquidity pool |
| `GET` | [`/account/:account/mempool`](#get-accountaccountmempool) | Show mempool transactions for account |
| `GET` | [`/account/:account/open_orders`](#get-accountaccountopen_orders) | Show all open orders for account |
| `GET` | [`/account/:account/open_orders/:asset`](#get-accountaccountopen_ordersasset) | Show open orders for account and specific asset |
| `GET` | [`/account/:account/balance/:tokenspec`](#get-accountaccountbalancetokenspec) | Show balance of specific account for a token |
| `GET` | [`/account/:account/wart_balance`](#get-accountaccountwart_balance) | Show WART balance of specific account |
| `GET` | [`/account/:account/history/:beforeTxIndex`](#get-accountaccounthistorybeforetxindex) | Show transaction history of specific account |
| `GET` | [`/account/richlist/:tokenspec`](#get-accountrichlisttokenspec) | Show richlist for a specific token |
| `GET` | [`/peers/ip_count`](#get-peersip_count) | Show peer IP counts |
| `GET` | [`/peers/banned`](#get-peersbanned) | Show banned peers |
| `GET` | [`/peers/offenses/:page`](#get-peersoffensespage) | Show offenses of peers |
| `GET` | [`/peers/connected/connection`](#get-peersconnectedconnection) | Show connection info of connected peers |
| `GET` | [`/peers/connection_schedule`](#get-peersconnection_schedule) | Show connection schedule |
| `GET` | [`/peers/unban`](#get-peersunban) | Unban all peers |
| `GET` | [`/peers/connected`](#get-peersconnected) | Show info of connected peers |
| `GET` | [`/peers/disconnect/:id`](#get-peersdisconnectid) | Disconnect a specific peer |
| `GET` | [`/peers/throttled`](#get-peersthrottled) | Show throttled peers |
| `GET` | [`/peers/transmission_hours`](#get-peerstransmission_hours) | Show transmission data by hours |
| `GET` | [`/peers/transmission_minutes`](#get-peerstransmission_minutes) | Show transmission data by minutes |
| `GET` | [`/chart/candles/:asset/:interval?from=...&to=...&n=...`](#get-chartcandlesassetinterval) | Show OHLCV candle data for a timeframe (5m, 1h, 1d) |
| `GET` | [`/chart/trades/:asset?from=...&to=...&n=...`](#get-charttradesasset) | Show trade data |
| `GET` | [`/chart/hashrate/block/:from/:to/:window`](#get-charthashrateblockfromtowindow) | Show hashrate chart by block range |
| `GET` | [`/chart/hashrate/time/:from/:to/:interval`](#get-charthashratetimefromtointerval) | Show hashrate chart by time range |
| `GET` | [`/tools/encode16bit/from_e8/:feeE8`](#get-toolsencode16bitfrom_e8feee8) | Round raw 64 integer to closest 16 bit representation |
| `GET` | [`/tools/encode16bit/from_string/:string`](#get-toolsencode16bitfrom_stringstring) | Round coin amount string to closest 16 bit representation |
| `GET` | [`/tools/parse_price/:price/:decimals`](#get-toolsparse_pricedecimals) | Parse price adjusted for asset precision |
| `GET` | [`/tools/info`](#get-toolsinfo) | Print information about this node |
| `GET` | [`/tools/wallet/new`](#get-toolswalletnew) | Create a new wallet |
| `GET` | [`/tools/wallet/from_privkey/:privkey`](#get-toolswalletfrom_privkeyprivkey) | Restore wallet from a private key |
| `GET` | [`/tools/janushash_number/:headerhex`](#get-toolsjanushash_numberheaderhex) | Show number interpretation of a header's Janushash |
| `GET` | [`/tools/sample_verified_peers/:number`](#get-toolssample_verified_peersnumber) | List verified peers |
| `GET` | [`/debug/header_download`](#get-debugheader_download) | Show header download status |
| `GET` | [`/loadtest/block_request/:conn_id`](#get-loadtestblock_requestconn_id) | Load test block request |
| `GET` | [`/loadtest/header_request/:conn_id`](#get-loadtestheader_requestconn_id) | Load test header request |
| `GET` | [`/loadtest/disable/:conn_id`](#get-loadtestdisableconn_id) | Disable load test connection |
| `GET` | [`/debug/fakemine`](#get-debugfakemine) | Generate fake mining data (default address) |
| `GET` | [`/debug/rollback`](#get-debugrollback) | Rollback chain |
| `GET` | [`/debug/fakemine/:address`](#get-debugfakemineaddress) | Generate fake mining data for address |

For WebSocket API documentation, see [WebSocket API](WebSocket.md).

## Detailed Description

### `POST /transaction/add`

Create new transactions as described [here](rest/create-transaction.md). Returns transaction hash in hex format:

### `GET /transaction/mempool`

Show content of mempool. Example output:

```json
{
 "code": 0,
 "data": [
  {
   "tag": "wart_transfer",
   "transaction": {
    "data": { "amount": { "E8": 100000000, "str": "1.00000000" }, "toAddress": "8733d0e..." },
    "hash": "f364da997bf7a3c3...",
    "signedCommon": { "fee": { "E8": 9992, "str": "0.00009992" }, "nonceId": 0, "originAddress": "2de77d5e...", "originId": 12345, "pinHeight": 0 }
   }
  }
 ]
}
```

### `GET /transaction/lookup/:txid`

Transaction lookup by transaction id. Example output of `/transaction/lookup/4b3bc48295742b71ff7c3b98ede5b652fafd16c67f0d2db6226e936a1cdbf0a5`:

```json
{
 "code": 0,
 "data": {
  "type": "Reward",
  "confirmations": 8,
  "mined": {
   "blockHeight": 376696,
   "timestamp": 1695472249,
   "utc": "2023-09-23 12:30:49 UTC"
  },
  "transaction": {
   "data": { "amount": { "E8": 300000000, "str": "3.00000000" }, "toAddress": "848b08b..." },
   "hash": "4b3bc48295742b71ff7c3b98ede5b652fafd16c67f0d2db6226e936a1cdbf0a5",
   "signedCommon": { "fee": { "E8": 0, "str": "0" }, "nonceId": 0, "originAddress": "00000000...", "originId": 376695, "pinHeight": 0 }
   }
 }
}

### `GET /transaction/latest`

Show latest transactions. Returns the most recent blocks with their actions.

**Output:**
- `fromId` — the lowest history ID in the result set
- `perBlock` — array of blocks, each containing `height`, `confirmations`, and `actions` (see [!ref Block Actions](rest/block-actions.md))

Example output of `/transaction/latest`:

```json
{
 "code": 0,
 "data": {
  "perBlock": [
   {
    "height": 21,
    "confirmations": 4,
    "actions": {
     "reward": {
      "historyId": 39,
      "transaction": {
       "data": { "amount": { "E8": 300039969, "str": "3.00039969" }, "toAddress": "00000000..." },
       "hash": "91b5330d1a3f3c9..."
      }
     },
     "wartTransfers": [
      {
       "historyId": 40,
       "transaction": {
        "data": { "amount": { "E8": 100000000, "str": "1.00000000" }, "toAddress": "00000000..." },
        "hash": "897b3cc72950b41e...",
        "signedCommon": { "fee": { "E8": 9992, "str": "0.00009992" }, "nonceId": 0, "originAddress": "3661579d...", "originId": 12344, "pinHeight": 0 }
       }
      }
     ],
     "tokenTransfers": [
      {
       "historyId": 41,
       "transaction": {
        "data": { "amount": { "str": "99.9912", "u64": 999912, "decimals": 4 }, "asset": { "hash": "f45b1131...", "id": 2, "name": "TOK2", "decimals": 4 }, "isLiquidity": false, "toAddress": "00000000...", "tokenSpec": "asset:f45b1131..." },
        "hash": "87a8f53490f32a2...",
        "signedCommon": { "fee": { "E8": 9992, "str": "0.00009992" }, "nonceId": 1, "originAddress": "3661579d...", "originId": 12344, "pinHeight": 0 }
       }
      }
     ],
     "newOrders": [...],
     "matches": [...],
     "liquidityDeposits": [...],
     "liquidityWithdrawals": [...],
     "assetCreations": [...],
     "cancelations": [...]
    }
   },
   {
    "height": 8,
    "confirmations": 17,
    "actions": {
     "reward": {
      "historyId": 11,
      "transaction": {
       "data": { "amount": { "E8": 300009992, "str": "3.00009992" }, "toAddress": "00000000..." },
       "hash": "576da5fde183f4b..."
      }
     },
     "wartTransfers": [],
     "tokenTransfers": [],
     "newOrders": [],
     "matches": [],
     "liquidityDeposits": [
      {
       "historyId": 12,
       "transaction": {
        "data": { "baseAsset": { "hash": "f45b1131...", "id": 2, "name": "TOK2", "decimals": 4 }, "deposited": { "base": { "str": "99.9912", "u64": 999912, "decimals": 4 }, "quote": { "str": "0.00999912", "E8": 999912 } },
        "hash": "a3f6d1d3ee13cb8...",
        "processed": { "sharesReceived": { "str": "99.9912", "u64": 999912, "decimals": 4 } },
        "signedCommon": { "fee": { "E8": 9992, "str": "0.00009992" }, "nonceId": 25, "originAddress": "3661579d...", "originId": 12344, "pinHeight": 0 }
       }
      }
     ],
     "liquidityWithdrawals": [],
     "assetCreations": [...],
     "cancelations": []
    }
   },
   ...
  ],
  "fromId": 1
 }
}
```

For detailed JSON structure of each action type, see [!ref Block Actions](rest/block-actions.md).
### `GET /transaction/minfee`

Show the minimum fee required by this node. Transactions with a lower fee will not be accepted or requested by the node.

Example output:

```json
{
 "code": 0,
 "data": {
  "minFee": {
   "E8": 9992,
   "str": "0.00009992",
   "bytes": "2722"
  }
 }
}

### `GET /dex/market/:market`

Show market orders and liquidity pool for a specific asset. The `:market` parameter is the asset identifier.

Example output of `/dex/market/7`:

```json
{
 "code": 0,
 "data": {
  "baseAsset": { "decimals": 4, "hash": "0e4825ef...", "id": 7, "name": "TOK2" },
  "wartToAssetSwaps": [
   {
    "amount": { "str": "100.0000", "u64": 1000000, "decimals": 4 },
    "filled": { "str": "0", "u64": 0, "decimals": 4 },
    "inMempool": false,
    "limit": { "doubleAdjusted": 0.1, "doubleRaw": 1000.0, "exponent2": -6, "hex": "fa0049", "mantissa": 64000, "precExponent10": 4 },
    "txHash": "c6fbade2..."
   }
  ],
  "assetToWartSwaps": [...],
  "liquidityPool": {
   "asset": { "str": "1000.0000", "u64": 10000000, "decimals": 4 },
   "shares": { "str": "100.0000", "u64": 1000000, "decimals": 4 },
   "wart": { "E8": 100000000, "str": "1.00000000" }
  },
  "match": {
   "baseAsset": { "decimals": 4, "hash": "0e4825ef...", "id": 7, "name": "TOK2" },
   "buySwaps": [...],
   "poolAfter": { "base": { "str": "10.0000", "u64": 100000, "decimals": 4 }, "quote": { "str": "1.00000000", "E8": 100000000 } },
   "poolBefore": { "base": { "str": "0", "u64": 0, "decimals": 4 }, "quote": { "str": "0", "E8": 0 } },
   "sellSwaps": [...]
  }
 }
}
```

### `GET /account/:account/mempool`

Show mempool transactions for a specific account. Same structure as `GET /transaction/mempool` but filtered to this account.

```json
{
 "code": 0,
 "data": [
  {
   "tag": "wart_transfer",
   "transaction": {
    "data": { "amount": { "E8": 100000000, "str": "1.00000000" }, "toAddress": "8733d0e..." },
    "hash": "f364da997bf7a3c3...",
    "signedCommon": { "fee": { "E8": 9992, "str": "0.00009992" }, "nonceId": 0, "originAddress": "2de77d5e...", "originId": 12345, "pinHeight": 0 }
   }
  }
 ]
}
```

### `GET /account/:account/open_orders`

Show all open orders for a specific account across all markets.

```json
{
 "code": 0,
 "data": [
  {
   "baseAsset": { "decimals": 4, "hash": "0e4825ef...", "id": 7, "name": "TOK2" },
   "wartToAssetSwaps": [
    {
     "amount": { "str": "100.0000", "u64": 1000000, "decimals": 4 },
     "filled": { "str": "10.0000", "u64": 100000, "decimals": 4 },
     "inMempool": false,
     "limit": { "doubleAdjusted": 0.1, "doubleRaw": 1000.0, "exponent2": -6, "hex": "fa0049", "mantissa": 64000, "precExponent10": 4 },
     "txHash": "c6fbade2..."
    }
   ],
   "assetToWartSwaps": [...]
  }
 ]
}
```

### `GET /account/:account/open_orders/:asset`

Show open orders for a specific account and specific base asset. Same structure as above, filtered to one asset.

### `GET /account/:account/balance/:tokenspec`

Show balance of an account for a specific asset. The `:account` parameter is the account address, and `:tokenspec` is the token spec (e.g., `asset:0e4825efffa294610d2ac376713e3bcc9b53d378e823834b64e5df01f75d3b0c`).

 #### Balance Types

 The API returns three balance types:

 | Type | Description |
 |------|-------------|
 | `total` | The total balance of the account |
 | `locked` | Balance locked in open orders |
 | `mempool` | Balance currently used by pending transactions in the mempool |

 Spendable amount is `total - locked` and can be reused for new transactions. If `total - locked` is sufficient but `total - locked - mempool` is not, the system may evict lower-fee transactions from the mempool to make room for a higher-fee transaction. 

 Example output:

 ```json
 {
  "code": 0,
  "data": {
   "account": {
    "accountId": 9,
    "address": "2de77d5e23dc63e4c4149d394c979361e9e8e966336c8cd4"
   },
   "balance": {
    "locked": {
     "precision": 4,
     "str": "12479.9990",
     "u64": 124799990
    },
    "mempool": {
     "precision": 4,
     "str": "100.0000",
     "u64": 1000000
    },
    "total": {
     "precision": 4,
     "str": "30899.9999",
     "u64": 308999999
    }
   },
   "lookupTrace": {
    "fails": [],
    "snapshotHeight": null
   },
   "token": {
    "id": 15,
    "name": "TOK2",
    "precision": 4,
    "spec": "asset:0e4825efffa294610d2ac376713e3bcc9b53d378e823834b64e5df01f75d3b0c"
   }
  }
 }
```

### `GET /account/:account/wart_balance`

Show WART balance (locked, mempool and total) of a specific account. Amount in `mempool` cannot exceed `total - locked`.

Example output of `account/2de77d5e23dc63e4c4149d394c979361e9e8e966336c8cd4/wart_balance`:

```json
{
 "code": 0,
 "data": {
  "account": {
   "accountId": 9,
   "address": "2de77d5e23dc63e4c4149d394c979361e9e8e966336c8cd4"
  },
  "wart": {
   "locked": { "E8": 0, "str": "0" },
   "mempool": { "E8": 0, "str": "0" },
   "total": { "E8": 400097559, "str": "4.00097559" }
  }
 }
}
```

### `GET /account/:account/history/:beforeTxIndex`

Show transaction history of specific account. Returns blocks containing actions relevant to the account, structured as `perBlock[].actions` (see [Block Actions](rest/block-actions.md) for field details).

Example output:

```json
{
 "code": 0,
 "data": {
  "fromId": 69626,
  "perBlock": [
   {
     "actions": {
      "wartTransfers": [...],
      "tokenTransfers": [...],
      "newOrders": [...],
      "matches": [...],
      "liquidityDeposits": [...],
      "liquidityWithdrawals": [...],
      "assetCreations": [...],
      "cancelations": [...],
      "reward": null
     },
    "confirmations": 9949,
    "height": 366700
   }
  ]
 }
}
```

For detailed JSON structure of each action type, see [!ref Block Actions](rest/block-actions.md).

### `GET /account/richlist/:tokenspec`

Show richlist for a specific token. Example output of `/account/richlist/asset:0e4825ef...`:

```json
{
 "code": 0,
 "data": {
  "richlist": [
   {
    "address": "2de77d5e23dc63e4c4149d394c979361e9e8e966336c8cd4",
    "balance": { "str": "50000.0000", "u64": 500000000, "decimals": 4 }
   },
   {
    "address": "8733d0e21bc791f44785f02753bb589b8423eb56cd938fbc",
    "balance": { "str": "25000.0000", "u64": 250000000, "decimals": 4 }
   }
  ],
  "token": { "id": 7, "name": "TOK2", "precision": 4, "spec": "asset:0e4825ef..." }
 }
}
```

### `GET /peers/ip_count`

Show count of connections per IP address.

```json
{
 "code": 0,
 "data": [
  [ "51.75.21.134", 2 ]
 ]
}
```

### `GET /peers/banned`

Show list of banned peers. Example output of `/peers/banned`:

```json
{
 "code": 0,
 "data": [
  {
   "banuntil": 1773043430,
   "ip": "51.75.21.134",
   "reason": "Invalid POW"
  }
 ]
}
```

### `GET /peers/offenses/:page`

Show paginated list of peer offenses. Example output of `/peers/offenses/0`:

```json
{
 "code": 0,
 "data": [
  {
   "ip": "51.75.21.134",
   "offense": "Invalid POW",
   "time": 1772443430
  }
 ]
}
```

The `time` field is a Unix timestamp.

### `GET /peers/connected/connection`

Show connection information of connected peers.

```json
{
 "code": 0,
 "data": [
  {
   "ip": "51.75.21.134",
   "port": 9186,
   "since": 1772443430
  }
 ]
}
```

### `GET /peers/connection_schedule`

Show the connection schedule for peers.

```json
{
 "code": 0,
 "data": {
  "schedule": [...]
 }
}
```

### `GET /peers/unban`

Unban all banned peers.

```json
{ "code": 0 }
```

### `GET /peers/connected`

Show detailed information about connected peers. Example output of `/peers/connected`:

```json
{
 "code": 0,
 "data": [
  {
   "chain": {
    "descriptor": 4,
    "forkLower": 0,
    "forkUpper": 0,
    "grid": ["..."],
    "length": 1198931,
    "worksum": 1.008584641271857e+19,
    "worksumHex": "0x..."
   },
   "connection": {
    "ip": "51.75.21.134",
    "port": 9186,
    "since": 1772443430
   },
   "leaderPriority": {
    "ack": { "priority": 0, "since": 0 },
    "theirs": { "priority": 0, "since": 0 }
   },
   "throttle": {
    "blockRequest": { "priority": 0, "since": 0 },
    "delay": 0,
    "headerRequest": { "priority": 0, "since": 0 }
   }
  }
 ]
}
```

| Field | Type | Description |
|-------|------|-------------|
| `chain` | object | Chain state of the peer |
| `chain.grid` | array | Header grid for chain comparison |
| `chain.length` | integer | Chain height of the peer |
| `chain.worksum` | number | Total work in the chain |
| `connection` | object | Connection info |
| `connection.ip` | string | Peer IP address |
| `connection.port` | integer | Peer port |
| `connection.since` | integer | Unix timestamp of connection start |
| `leaderPriority` | object | Leader election priority |
| `throttle` | object | Throttling state for block/header requests |

### `GET /peers/disconnect/:id`

Disconnect a specific peer by connection ID.

```json
{ "code": 0 }
```

### `GET /peers/throttled`

Show throttled peers.

```json
{
 "code": 0,
 "data": [
  {
   "connection": { "endpoint": "51.75.21.134:9186", "id": 12345 },
   "throttle": {
    "blockRequest": { "priority": 0, "since": 0 },
    "delay": 5000,
    "headerRequest": { "priority": 0, "since": 0 }
   }
  }
 ]
}
```

### `GET /peers/transmission_hours`

Show transmission statistics aggregated by hours.

```json
{
 "code": 0,
 "data": {
  "byHost": {
   "51.75.21.134:9186": [
    { "rxBytes": 1024000, "timestamp": 1772443430, "txBytes": 512000 },
    { "rxBytes": 2048000, "timestamp": 1772447030, "txBytes": 1024000 }
   ]
  }
 }
}
```

### `GET /peers/transmission_minutes`

Show transmission statistics aggregated by minutes. Same structure as `/peers/transmission_hours` but with per-minute granularity.

### `GET /tools/encode16bit/from_e8/:feeE8`

 Round raw fee integer representation (coin amount is this number divided by 10^8) to closest 16 bit representation. This is required for fee specification in the `/transaction/add` endpoint. Example output of `/tools/encode16bit/from_e8/5002`

```json
{
 "code": 0,
 "data": {
  "16bit": 12514,
  "originalAmount": "0.00005002",
  "originalE8": 5002,
  "roundedAmount": "0.00005000",
  "roundedE8": 5000
 }
}
```

### `GET /tools/encode16bit/from_string/:string`

 Round fee amount string to closest 16 bit representation. This is required for fee specification in the `/transaction/add` endpoint. Example output of `/tools/encode16bit/from_string/0.001`:

```json
{
 "code": 0,
 "data": {
  "16bit": 16922,
  "originalAmount": "0.00100000",
  "originalE8": 100000,
  "roundedAmount": "0.00099968",
  "roundedE8": 99968
 }
}
```

### `GET /tools/parse_price/:price/:decimals`

Parse price adjusted for asset precision.

Example output of `/tools/parse_price/0.123456/3`:

```json
{
 "code": 0,
 "data": {
  "assetPrecision": 3,
  "ceil": {
   "doubleAdjusted": 0.12345750000000001,
   "doubleRaw": 12345.75,
   "exponent2": -2,
   "hex": "c0e74d",
   "mantissa": 49383,
   "precExponent10": 5
  },
  "floor": {
   "doubleAdjusted": 0.12345500000000001,
   "doubleRaw": 12345.5,
   "exponent2": -2,
   "hex": "c0e64d",
   "mantissa": 49382,
   "precExponent10": 5
  }
 }
}
```

### `GET /tools/info`

Print information about this node.

Example output of `/tools/info`:

```json
{
 "code": 0,
 "data": {
  "chainDBPath": "/home/user/.warthog/chain_defi.db3",
  "dbSize": 3058028544,
  "peersDBPath": "/home/user/.warthog/peers_v2.db3",
  "rxtxDBPath": "/home/user/.warthog/rxtx.db3",
  "uptime": {
   "formatted": "0d 0h 6m 8s",
   "seconds": 368,
   "since": 1763453691
  },
  "version": {
   "commit": "dd054a2",
   "major": 0,
   "minor": 10,
   "name": "v0.10.3 \"dd054a2\"",
   "patch": 3
  }
 }
}
```

### `GET /tools/wallet/new`

 Create a new wallet.

 Example output of `/tools/wallet/new`:

 ```json
 {
  "code": 0,
  "data": {
   "address": "11c6c0f7148a845cc899360a38fa839ba3d5e719293bb5c6",
   "privKey": "d711913bea313c7a9007e115a5969d9e1127f119d7ec72b3d713f698ab591488",
   "pubKey": "03a907038c8aba55cecf71b242c18d4c92e2a37ff83b8779444f10ceb720fe4551"
  }
 }
```

!!!warning Warning
This endpoint should only be used for testing purposes. For security reasons the node should not be involved in wallet-related operations involving a private key.
!!!

### `GET /tools/wallet/from_privkey/:privkey`
Restore wallet from a private key.

Example output of `/tools/wallet/from_privkey/d3ce2210adf0fccabe31b61309e2b80c029a7e4e305aeed29432edd428d35c3d`:

```json
{
 "code": 0,
 "data": {
  "address": "31bfdc6b23b22130edc05073cffa29be793a32518769180e",
  "privKey": "d3ce2210adf0fccabe31b61309e2b80c029a7e4e305aeed29432edd428d35c3d",
  "pubKey": "022ad688a8ccc4898b2b923088e9f297785e8852a6bdc33b7e87ffea3d3e63e32b"
 }
}
```

### `GET /tools/janushash_number/:headerhex`
Show number interpretation of a header's Janushash. Mining corresponds to finding headers with this number smaller than some threshold dictated by the header difficulty.

Example output of `/tools/janushash_number/b4e91160d990b7b679234fc7cdc1aefbe11afe9823d6d8d1da7067e9ab0c8d380af405022613b56b379d0b071a4829cc27875f19bb0cb493f3f15aaedade1080e433874b0000000265ce0eb452f36e59`:

```json
{
 "code": 0,
 "data": {
  "janushashNumber": 4.315032866502006e-14
 }
}
```

### `GET /tools/sample_verified_peers/:number`
List verified peers.

Example output of `/tools/sample_verified_peers/5`:

```json
{
 "code": 0,
 "data": [
  "51.75.21.134:9186",
  "172.241.31.200:19110",
  "51.68.155.69:9186",
  "148.251.181.254:19110",
  "149.102.187.227:9186"
 ]
}
```

### `GET /debug/header_download`

Show header download status for debugging purposes. Example output:

```json
{
 "code": 0,
 "data": {
  "config": {
   "maxLeaders": 10,
   "pendingDepth": 10
  },
  "leaderListSize": 5,
  "minWork": "0x1d00ffff",
  "queuedBatches": {
   "51.75.21.134:9186": {
    "batchSize": 2000,
    "leaderRefsSize": 0,
    "originId": 12345,
    "probeRefsSize": 2000
   }
  },
  "verifierMapSize": 100
 }
}
```

### `GET /loadtest/block_request/:conn_id`

Load test endpoint for block requests.

```json
{ "code": 0 }
```

### `GET /loadtest/header_request/:conn_id`

Load test endpoint for header requests.

```json
{ "code": 0 }
```

### `GET /loadtest/disable/:conn_id`

Disable a load test connection.

```json
{ "code": 0 }
```

### `GET /debug/fakemine`

Generate fake mining data using the default address.

```json
{ "code": 0 }
```

### `GET /debug/rollback`

Rollback the chain by one block.

```json
{ "code": 0 }
```

### `GET /debug/fakemine/:address`

Generate fake mining data for a specific address.

```json
{ "code": 0 }
```

### `GET /chart/candles/:asset/:interval`

Show OHLCV candle data for a specific asset and interval.

**Parameters:**

| Parameter | Type | Description |
|-----------|------|-------------|
| `:asset` | integer/string | Asset ID (e.g. `7`) or hash (e.g. `0e4825efffa294610d2ac376713e3bcc9b53d378e823834b64e5df01f75d3b0c`)|
| `:interval` | string | Timeframe: `5m` (5 minutes), `1h` (1 hour), `1d` (1 day) |
| `from` | Unix timestamp | Start time (optional) |
| `to` | Unix timestamp | End time (optional) |
| `n` | integer | Limit number of results (optional, default: 100) |

**Query parameter rules:**
- Not all three optional parameters (`from`, `to`, `n`) can be specified at the same time.
- If only `from` is specified, the earliest entries from `from` are returned.
- If only `to` is specified, the latest entries up to `to` are returned.
- If `from` and `to` are specified, all entries from `from` to `to` are returned.
- The total number of entries is limited to 200. The default value for `n` is 100.

Example output of `/chart/candles/7/5m`:

```json
{
 "code": 0,
 "data": [
  [
   1772265900,          // begin timestamp of the candle
   7,                   // height
   0.09999999999999999, // open
   0.09999999999999999, // high
   0.09999999999999999, // low
   0.09999999999999999, // close
   40.00200000000001,   // asset amount (base)
   4.0002               // WART amount (quote)
  ]
 ]
}
```

Each candle is a 8-element array: `[begin_timestamp, height, open, high, low, close, base, quote]`.

### `GET /chart/trades/:asset`

Show trade data for a specific asset.

**Parameters:**

| Parameter | Type | Description |
|-----------|------|-------------|
| `:asset` | integer/string | Asset ID (e.g. `7`) or hash (e.g. `0e4825efffa294610d2ac376713e3bcc9b53d378e823834b64e5df01f75d3b0c`) |
| `from` | block height | Start block (optional) |
| `to` | block height | End block (optional) |
| `n` | integer | Limit number of results (optional, default: 100) |

**Query parameter rules:**
- Not all three optional parameters (`from`, `to`, `n`) can be specified at the same time.
- If only `from` is specified, the earliest entries from `from` are returned.
- If only `to` is specified, the latest entries up to `to` are returned.
- If `from` and `to` are specified, all entries from `from` to `to` are returned.
- The total number of entries is limited to 200. The default value for `n` is 100.

Example output of `/chart/trades/7`:

```json
{
 "code": 0,
 "data": [
  [
   7,                 // height
   1772266091,        // timestamp of the block containing the trade
   40.00200000000001, // total asset amount traded in the block
   4.0002             // total WART amount traded in the block
  ]
 ]
}
```

Each trade is a 4-element array: `[height, timestamp, base, quote]`.
