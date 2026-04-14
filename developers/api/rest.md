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
```
```

### `GET /transaction/latest`

Show latest transactions. Returns the most recent blocks with their full transaction details.

**Output:**
- `count` — total number of transactions returned
- `fromId` — the lowest history ID in the result set
- `perBlock` — array of blocks, each containing `header`, `body`, `confirmations`, `height`, `timestamp`, `utc`

Example output of `/transaction/latest`:

```json
{
 "code": 0,
 "data": {
  "count": 49,
  "fromId": 1,
  "perBlock": [
   {
    "actions": {
     "assetCreations": [...],
     "cancelations": [...],
     "liquidityDeposits": [...],
     "liquidityWithdrawals": [...],
     "matches": [...],
     "newOrders": [...],
     "orderCancelations": [...],
     "wartTransfers": [...],
     "tokenTransfers": [...],
     "reward": null
    },
    "confirmations": 1,
    "height": 23
   },
   ...
  ]
 }
}
```

For detailed JSON structure of block actions, see [!ref Block Actions](rest/block-actions.md)
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
```

### `GET /settings/mempool/minfee/:feeE8`

 Adjust the minimum transaction fee. The desired new minimum transaction fee should be specified.

 Exemple output:

 ```json
 {
  "code": 0,
  "data": {
   "deleted": 0
  }
 }
```

 You can also set the min fee when launching your node.

 Exemple:

 ```bash
 $./wart-node --minfee="0.00009992"
 ```


### `GET /chain/head`

Show info on chain head. Example output:

```json
{
 "code": 0,
 "data": {
  "chainHead": {
   "difficulty": 30866431555762.105,
   "hash": "e9dfdcfdeec3c5376dd682e09cce909dc255d611e9cb36974bcbcbcecb6fb432",
   "hashrate": 296429051797,
   "height": 1198931,
   "is_janushash": true,
   "pinHash": "a7bd097d9c10dc393239f169f6dd5352e0b4e28e942aaa78b54c57ed1ec7315b",
   "pinHeight": 1198912,
   "worksum": 1.008584641271857e+19,
   "worksumHex": "0x0000000000000000000000000000000000000000000000008bf81fe0113cf6a0"
  },
  "synced": true
 }
}
```

### `GET /chain/grid`

Show hexadecimal header grid. This grid is used for chain sync to allow nodes spot points where chains diverge. Example output (truncated):

```json
{
 "code": 0,
 "data": {
  "headers": [
   "19f65d40c6bc02a16378712bd7652eaebc925955e96c0262641314520000000021c88fbd99f0d3369f7f1ad981e11b8c6f032e13d4ef28311226ea5c8574c3fe3635b32500000001649f78127c04d23d",
   "0e43cf390ca986fabbed37a33386b4ad73723d51fab291d12013351f0000000021d1c6076e52222fb8c83fe33d55c610482cd272b7789646710510df3ba37a4f55b30e4e0000000164a221eb03577627",
   "..."
  ]
 }
}
 ```

### `GET /chain/block/:id/hash`

 Show hash of specific block. Example output of `/chain/block/366700/hash`

 ```json
{
 "code": 0,
 "data": {
  "hash": "ac4f50ef90c0730075f4995d0b33691e9e7d69f84d2191c029371c0000000000"
 }
}
```

### `GET /chain/block/:id/header`

 Show header of specific block.

 Example output of `/chain/block/366700/header`

 ```json
 {
 "code": 0,
 "data": {
  "header": {
   "difficulty": 5850035871080.048,
   "hash": "ac4f50ef90c0730075f4995d0b33691e9e7d69f84d2191c029371c0000000000",
   "merkleroot": "a4d44aff9ebd539f2b5e758fb76df22bfd13a2e4d894752c7892f9e9c3f3d969",
   "nonce": "0d8e8eab",
   "prevHash": "11f2b6860910f7b544e01fd5ed5419588c29e42b305553bd492a0d0000000000",
   "raw": "11f2b6860910f7b544e01fd5ed5419588c29e42b305553bd492a0d00000000002ac075d9a4d44aff9ebd539f2b5e758fb76df22bfd13a2e4d894752c7892f9e9c3f3d96900000001650bd0820d8e8eab",
   "target": "d975c02a",
   "timestamp": 1695273090,
   "utc": "2023-09-21 05:11:30 UTC",
   "version": "00000001"
  }
 }
}
```

### `GET /chain/block/:id/binary`

Show binary data of specific block with structure annotations.

Example output of `/chain/block/20/binary`:

```json
{
 "code": 0,
 "data": {
  "bytes": "00000000000000000000000000000000000000040000000011e1a30100000000000000010000000000000007000000040000000000000000000009000000630000000000000000000000000f4240fa004902fc0829a8a7fa7b7016935f3bf5e8db4c5aed491dd6c0609180133ce5cff48c3134d287f28f0a07b266f68f04ddec146a93f9237a8f779dee0d2f7fc90b2d56010000",
  "structure": [
   {
    "children": [],
    "offsetBegin": 0,
    "offsetEnd": 10,
    "tag": "extraNonce"
   },
   {
    "children": [
     {
      "children": [],
      "offsetBegin": 10,
      "offsetEnd": 12,
      "tag": "length"
     }
    ],
    "offsetBegin": 10,
    "offsetEnd": 12,
    "tag": "newAddresses"
   },
   {
    "children": [
     {
      "children": [],
      "offsetBegin": 12,
      "offsetEnd": 20,
      "tag": "toAccountId"
     },
     {
      "children": [],
      "offsetBegin": 20,
      "offsetEnd": 28,
      "tag": "wart"
     }
    ],
    "offsetBegin": 12,
    "offsetEnd": 28,
    "tag": "reward"
   },
   {
    "children": [
     {
      "children": [],
      "offsetBegin": 28,
      "offsetEnd": 32,
      "tag": "length"
     }
    ],
    "offsetBegin": 28,
    "offsetEnd": 32,
    "tag": "wartTransfers"
   },
   {
    "children": [
     {
      "children": [],
      "offsetBegin": 32,
      "offsetEnd": 34,
      "tag": "length"
     }
    ],
    "offsetBegin": 32,
    "offsetEnd": 34,
    "tag": "cancelations"
   },
   {
    "children": [
     {
      "children": [],
      "offsetBegin": 34,
      "offsetEnd": 36,
      "tag": "length"
     },
     {
      "children": [],
      "offsetBegin": 36,
      "offsetEnd": 44,
      "tag": "assetId"
     },
     {
      "children": [],
      "offsetBegin": 44,
      "offsetEnd": 51,
      "tag": "tenBitsLengths"
     },
     {
      "children": [
       {
        "children": [],
        "offsetBegin": 51,
        "offsetEnd": 59,
        "tag": "originAccountId"
       },
       {
        "children": [],
        "offsetBegin": 59,
        "offsetEnd": 67,
        "tag": "pinNonce"
       },
       {
        "children": [],
        "offsetBegin": 67,
        "offsetEnd": 69,
        "tag": "compactFee"
       },
       {
        "children": [],
        "offsetBegin": 70,
        "offsetEnd": 78,
        "tag": "amount"
       },
       {
        "children": [],
        "offsetBegin": 78,
        "offsetEnd": 81,
        "tag": "limitPrice"
       },
       {
        "children": [],
        "offsetBegin": 81,
        "offsetEnd": 146,
        "tag": "signature"
       }
      ],
      "offsetBegin": 51,
      "offsetEnd": 146,
      "tag": "limitSwap"
     }
    ],
    "offsetBegin": 34,
    "offsetEnd": 146,
    "tag": "tokenSections"
   },
   {
    "children": [
     {
      "children": [],
      "offsetBegin": 146,
      "offsetEnd": 148,
      "tag": "length"
     }
    ],
    "offsetBegin": 146,
    "offsetEnd": 148,
    "tag": "assetCreations"
   }
  ]
 }
}
```

### `GET /chain/block/:id`

 Show header and body of specific block.
 Example output of `/chain/block/366700`

 ```json
 {
 "code": 0,
 "data": {
  "body": {
   "rewards": [
    {
     "amount": "3.00000001",
     "amountE8": 300000001,
     "toAddress": "848b08b803e95640f8cb30af1b3166701b152b98c2cd70ee",
     "txHash": "660b6f0883e4b295d5a3920eea9c73636787b932efd00cacc6d3a9a89f540714"
    }
   ],
   "transfers": [
    {
     "amount": "5000.00000000",
     "amountE8": 500000000000,
     "fee": "0.00000001",
     "feeE8": 1,
     "fromAddress": "1711cadfe5a66a5f5f245fc78b2335f79da362178a72de2e",
     "nonceId": 131630562,
     "pinHeight": 366688,
     "toAddress": "8733d0e21bc791f44785f02753bb589b8423eb56cd938fbc",
     "txHash": "f364da997bf7a3c3bc8ead22041509228d6bdea39fcbcdc6be56030433d54219"
    }
   ]
  },
  "confirmations": 9970,
  "header": {
   "difficulty": 5850035871080.048,
   "hash": "ac4f50ef90c0730075f4995d0b33691e9e7d69f84d2191c029371c0000000000",
   "merkleroot": "a4d44aff9ebd539f2b5e758fb76df22bfd13a2e4d894752c7892f9e9c3f3d969",
   "nonce": "0d8e8eab",
   "prevHash": "11f2b6860910f7b544e01fd5ed5419588c29e42b305553bd492a0d0000000000",
   "raw": "11f2b6860910f7b544e01fd5ed5419588c29e42b305553bd492a0d00000000002ac075d9a4d44aff9ebd539f2b5e758fb76df22bfd13a2e4d894752c7892f9e9c3f3d96900000001650bd0820d8e8eab",
   "target": "d975c02a",
   "timestamp": 1695273090,
   "utc": "2023-09-21 05:11:30 UTC",
   "version": "00000001"
  },
  "height": 366700,
  "timestamp": 1695273090,
  "utc": "2023-09-21 05:11:30 UTC"
 }
}
```

### `GET /chain/mine/:account`

 Generate data required for mining. Example output of `/chain/mine/11c6c0f7148a845cc899360a38fa839ba3d5e719293bb5c6`

```json
{
 "code": 0,
 "data": {
  "blockReward": "3.00000000",
  "blockRewardE8": 300000000,
  "body": "000000000000000000000001e94f274f08cf64bd76688ffb34012269e045b737000000000000098d0000000011e1a300",
  "difficulty": 30866431555762.105,
  "header": "50eaaf1453c84cabef749af9064d73d881174a44d7f165122400604e71342f9d0b2479fd6ec2b44e00bed326c9116b2f85f9f72bdd2157cc521cf49d0cf360fad91b881e00000002660ab00400000000",
  "height": 1198924,
  "merklePrefix": "72aa3aa84afd92af788b750d28ed9fe247d8788956bbdddc5a92cbdcb5925e4fbe20dcff8dc31364a524061ff0235bd78c9786d6d29dd2a73b49efacb57966d6",
  "synced": true,
  "testnet": false,
  "totalTxFee": "0",
  "totalTxFeeE8": 0
 }
}```

### `GET /chain/txcache`

 Show transaction cache. Example output:

 ```json
{
 "code": 0,
 "data": []
 }
 ```

### `GET /chain/hashrate/:window`

 Show current hashrate based on latest `n` blocks

Example output of `/chain/hashrate/100`

 ```json
{
 "code": 0,
 "data": {
  "estimate": 296429051797,
  "nBlocks": 100
 }
}
```

### `GET /chain/signed_snapshot`

Show chain snapshot. Example output:

```json
{
 "code": 0,
 "data": {
  "hash": "a7bd097d9c10dc393239f169f6dd5352e0b4e28e942aaa78b54c57ed1ec7315b",
  "priorityHeight": 1198912,
  "priorityImportance": 12345,
  "signature": "..."
 }
}
}
 ```

### `GET /chart/hashrate/block/:from/:to/:window`

Show hashrate chart data for a block range. The `:from` and `:to` parameters are block heights, and `:window` is the averaging window.

Example output of `/chart/hashrate/block/100/200/10`:

```json
{
 "code": 0,
 "data": {
  "data": [296429051797.5, 288123456789.0, ...],
  "range": { "begin": 100, "end": 200 }
 }
}
```

Each entry in `data` is the estimated hashrate for a block, averaged over the specified window.

### `GET /chart/hashrate/time/:from/:to/:interval`

Show hashrate chart data for a time range. The `:from` and `:to` parameters are block heights, and `:interval` is the time interval in seconds.

Example output of `/chart/hashrate/time/100/200/3600`:

```json
{
 "code": 0,
 "data": {
  "data": [
   { "height": 100, "hashrate": 296429051797, "timestamp": 1772443430 },
   { "height": 101, "hashrate": 288123456789, "timestamp": 1772443490 }
  ],
  "interval": 3600,
  "range": { "begin": 100, "end": 200 }
 }
}
```

Each entry in `data` is `{ height, hashrate, timestamp }` for the beginning of the interval.

### `POST /chain/append`

 Append mined block. Miners must POST mined block they received from `GET /chain/mine/:address` with mined nonce (in header) and extranonce (first 10 bytes in block).
 Output on success: `{"code": 0, data: null}`, example on error: `{"code": <error code>, error: "<error message>"}`.

### `GET /asset/complete?namePrefix=...&hashPrefix=...`

Search assets by query parameters `namePrefix` (asset name prefix) and `hashPrefix` (hash prefix) especially suitable for auto-complete.
Example output of "/asset/complete?namePrefix=TOK"
```json
{
 "code": 0,
 "data": {
  "hashPrefix": "",
  "matches": [
   {
    "decimals": 4,
    "hash": "0e4825efffa294610d2ac376713e3bcc9b53d378e823834b64e5df01f75d3b0c",
    "height": 7,
    "name": "TOK2"
   },
   {
    "decimals": 4,
    "hash": "58a74b758516d13e7f882922b483fca1627758c85b491e2cb9303e92e270ba88",
    "height": 4,
    "name": "TOK2"
   },
   {
    "decimals": 4,
    "hash": "d645efa5c0a64b409e44725d1d9b84ed7d800fca8ca5b3d71301d361652eddbc",
    "height": 22,
    "name": "TOK2"
   },
   {
    "decimals": 4,
    "hash": "f45b113119c7f7c000234f1090d5d181ab60b8b24526f1edd2f563aa1ca329f2",
    "height": 3,
    "name": "TOK2"
   }
  ],
  "namePrefix": "TOK"
 }
}
```

### `GET /asset/lookup/:asset`

Look up asset information by asset ID or asset hash.

Example output of `/asset/lookup/7`:

```json
{
 "code": 0,
 "data": {
  "decimals": 4,
  "groupId": 0,
  "hash": "0e4825efffa294610d2ac376713e3bcc9b53d378e823834b64e5df01f75d3b0c",
  "height": 7,
  "id": 7,
  "name": "TOK2",
  "ownerAccountId": 2,
  "parentId": null,
  "totalSupply": "100000.0000"
 }
}
```
```

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
     "orderCancelations": [...],
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
