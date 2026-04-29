# REST API

!!!
This documentation was partially generated with AI augmentation. Report inconsistencies at [github.com/warthog-network/docs/issues](https://github.com/warthog-network/docs/issues).
!!!

API for Warthog node version v0.10.3 "996c267"

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
| `GET` | [`/chain/block/:height/hash`](#get-chainblockheighthash) | Show hash of specific block |
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
| `GET` | [`/account/:account/history/:beforeId`](#get-accountaccounthistorybeforeid) | Show transaction history of specific account |
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

For now  [WebSocket API](WebSocket.md) is temporarily disabled.

## JSON Schemas

The Warthog node supports an automatically generated JSON Schema output of all output properties available at `/debug/json_schemas`. This can be especially useful in connection with AI coding assistants.

## Detailed Description

### `POST /transaction/add`

Create new transactions as described [here](rest/create-transaction.md). Returns transaction hash in hex format:
```json
{
 "code": 0,
 "txHash": "4c3bc48295742b71ff7c3b98ede5b652fafd16c67f0d2db6226e936a1cdbf0e5"
}
```

### `GET /transaction/mempool`

Show content of mempool as an array transactions. Each transaction is returned in the form
```json
{
 "type": "<string>",     // type of the transaction
 "transaction": {
  "data": ...,           // transaction specific data
  "hash": "f364da99...", // transaction hash
  "signedCommon": ...    // signature related data
 }
}
```
The content of the `data` field depends on the transaction type specified in the "type" field. See [Transactions](./rest/transactions.md) for details.

Note that the mempool contains only transactions that were not yet mined, such that there are no [implicit transactions](./rest/transactions.md#implicit-transactions). This is why all returned transactions are signed (i.e. have a `signedCommon` field) and don't contain a `processed` field.

==- Example Output:
```json
{
 "code": 0,
 "data": [
  {
   "type": "wartTransfer",
   "transaction": {
    "data": {
     "toAddress": "0000000000000000000000000000000000000000de47c9b2",
     "amount": {
      "str": "1.00000000",
      "E8": 100000000
     }
    },
    "hash": "dcb7959bcbc9403757121e491bee3cddc647271152c505913aac8b3b0d2c5a17",
    "signedCommon": {
     "originId": 9,
     "originAddress": "2de77d5e23dc63e4c4149d394c979361e9e8e966",
     "fee": {
      "str": "1.99884800",
      "E8": 199884800
     },
     "nonceId": 13311,
     "pinHeight": 0
    }
   }
  }
 ]
}
```
===

### `GET /transaction/lookup/:txid`

Transaction lookup by transaction id.
```json
{
 "confirmations": 1      // number of block confirmations of this transaction, 0 if in mempool
 "mined": ...,           // null or details on history id and block
 "type": "<string>",     // type of the transaction
 "transaction": {
  "data": ...,           // transaction specific data
  "hash": "f364da99...", // transaction hash
  "processed": ...,      // this field is present only for some transaction types, holds null or processed data
  "signedCommon": ...    // signature related data
 }
}
```
Compared to returned structure of [`/transaction/mempool`](#get-transactionmempool), here we have the additional fields `confirmations`, `mined` and for some transaction types `processed`.
The content of the `data` field depends on the transaction type specified in the `type` field. For some transaction types there exists the field `processed` which is null if and only if `mined` is null, i.e. if the transaction is still in mempool. See [Transactions](./rest/transactions.md) for details. If the transaction is already mined, `confirmations` will be non-zero and the `mined` field will be an object of structure
```json
{
 historyId: 7,           // history ID
 block: {                // details on the block where the transaction was mined
  height: 11,            // block height
  hash: "363a...89d5",   // block hash
  timestamp: 1771583754  // block timestamp
 }
}
```

==- Example output

```json
{
 "code": 0,
 "data": {
  "transaction": {
   "data": {
    "toAddress": "0000000000000000000000000000000000000000de47c9b2",
    "amount": {
     "str": "3.00000001",
     "E8": 300000001
    }
   },
   "hash": "50f860f2c626c002aaac9db9b1db566ba280b10fc24fa470728b0b4ed88008f9"
  },
  "type": "reward",
  "mined": {
   "historyId": 17,
   "block": {
    "height": 11,
    "hash": "363ac2530f8c38708dc313c9dfd9cddaa884fe29cf814b7bc316fed5298989d5",
    "timestamp": 1771583754
   }
  },
  "confirmations": 15
 }
}
```
===

### `GET /transaction/latest`

Show latest transactions. Returns the most recent blocks with their actions. On the top level the strucutre of returned data is
```json
{
 "fromId": 1       // the earliest transaction id returned
 "perBlock": [...] // array of transactions embedded in block structure
}
```
Each block in the returned `perBlock` array has the structure as returned by [`/chain/block/:id`](#get-chainblockid)
```json
{
 "confirmations": ... ,
 "header": {...}, // block header as in /chain/block/:id/header
 "height": ...,
 "body": {
  "reward": ... // null or reward
  "wartTransfer": [...],
  "tokenTransfer": [...],
  "limitSwap": [...],
  "match": [...],
  "liquidityDeposit": [...],
  "liquidityWithdrawal": [...],
  "assetCreation": [...],
  "cancelation": [...]
 }
}
```
All entries above in `body` are of the structure `{ "historyId": ..., "transaction": ...}` where `historyId` is an unsigned 64 bit integer and `transaction` has the structure
```json
{
 "data": ...,           // transaction specific data
 "hash": "f364da99...", // transaction hash
 "processed": ...,      // this field is present only for some transaction types, holds processed data
 "signedCommon": ...    // signature related data
}
```
See [Transactions](./rest/transactions.md) for details. In contrast to the `/transaction/lookup/:txid` endpoint, the `processed` field, if it is present for the respective transaction type, can never be `null` because we return mined transactions in this endpoint whereas the transaction returned by `/transaction/lookup/:txid` may still be in mempool which is exactly the case where `null` is returned for `processed`.

==- Example output
```json
{
 "code": 0,
 "data": {
  "perBlock": [
   {
    "height": 1,
    "confirmations": 25,
    "body": {
     "reward": {
      "transaction": {
       "data": {
        "toAddress": "3661579d61abde5837a8686dc4d65348a2fc61b1fe5f4093",
        "amount": {
         "str": "3.00000000",
         "E8": 300000000
        }
       },
       "hash": "da3dcfbc0dcb60be544a8510589f973b82651b48ddcb7943718dc3dcf93ae234"
      },
      "historyId": 1
     },
     "wartTransfer": [],
     "tokenTransfer": [],
     "limitSwap": [],
     "match": [],
     "liquidityDeposit": [],
     "liquidityWithdrawal": [],
     "assetCreation": [],
     "cancelation": []
    }
   },
   {
    "height": 2,
    "confirmations": 24,
    "body": {
     "reward": {
      "transaction": {
       "data": {
        "toAddress": "3661579d61abde5837a8686dc4d65348a2fc61b1fe5f4093",
        "amount": {
         "str": "3.00000000",
         "E8": 300000000
        }
       },
       "hash": "c870e807a46e6a5cf756165f95d511c7270a7dcd4eb51ddf7df047de3cfa1d8b"
      },
      "historyId": 2
     },
     "wartTransfer": [],
     "tokenTransfer": [],
     "limitSwap": [],
     "match": [],
     "liquidityDeposit": [],
     "liquidityWithdrawal": [],
     "assetCreation": [],
     "cancelation": []
    }
   },
   {
    "height": 3,
    "confirmations": 23,
    "body": {
     "reward": {
      "transaction": {
       "data": {
        "toAddress": "3661579d61abde5837a8686dc4d65348a2fc61b1fe5f4093",
        "amount": {
         "str": "3.00009992",
         "E8": 300009992
        }
       },
       "hash": "49d16e7f40c9ca4d681bf00569e9d5ede9ad006d0123c7bdb665434e5c7bf03d"
      },
      "historyId": 3
     },
     "wartTransfer": [],
     "tokenTransfer": [],
     "limitSwap": [],
     "match": [],
     "liquidityDeposit": [],
     "liquidityWithdrawal": [],
     "assetCreation": [
      {
       "transaction": {
        "data": {
         "name": "TOK2",
         "supply": {
          "str": "100000000.0000",
          "u64": 1000000000000,
          "decimals": 4
         }
        },
        "processed": {
         "assetId": 2
        },
        "hash": "f45b113119c7f7c000234f1090d5d181ab60b8b24526f1edd2f563aa1ca329f2",
        "signedCommon": {
         "originId": 1,
         "originAddress": "3661579d61abde5837a8686dc4d65348a2fc61b1",
         "fee": {
          "str": "0.00009992",
          "E8": 9992
         },
         "nonceId": 11,
         "pinHeight": 0
        }
       },
       "historyId": 4
      }
     ],
     "cancelation": []
    }
   },
   {
    "height": 4,
    "confirmations": 22,
    "body": {
     "reward": {
      "transaction": {
       "data": {
        "toAddress": "0000000000000000000000000000000000000000de47c9b2",
        "amount": {
         "str": "3.00009992",
         "E8": 300009992
        }
       },
       "hash": "5174ff690e763de12d212bf5a8cba8d2b58d5aa5545dbd7e28ed9e6b18f24982"
      },
      "historyId": 5
     },
     "wartTransfer": [],
     "tokenTransfer": [],
     "limitSwap": [],
     "match": [],
     "liquidityDeposit": [],
     "liquidityWithdrawal": [],
     "assetCreation": [
      {
       "transaction": {
        "data": {
         "name": "TOK2",
         "supply": {
          "str": "100000000.0000",
          "u64": 1000000000000,
          "decimals": 4
         }
        },
        "processed": {
         "assetId": 5
        },
        "hash": "58a74b758516d13e7f882922b483fca1627758c85b491e2cb9303e92e270ba88",
        "signedCommon": {
         "originId": 1,
         "originAddress": "3661579d61abde5837a8686dc4d65348a2fc61b1",
         "fee": {
          "str": "0.00009992",
          "E8": 9992
         },
         "nonceId": 12,
         "pinHeight": 0
        }
       },
       "historyId": 6
      }
     ],
     "cancelation": []
    }
   },
   {
    "height": 5,
    "confirmations": 21,
    "body": {
     "reward": {
      "transaction": {
       "data": {
        "toAddress": "0000000000000000000000000000000000000000de47c9b2",
        "amount": {
         "str": "3.00000000",
         "E8": 300000000
        }
       },
       "hash": "788f920a7eac73f1b3ad7701b3d9e4972e795a164b7f270f68c8f962ae17cf33"
      },
      "historyId": 7
     },
     "wartTransfer": [],
     "tokenTransfer": [],
     "limitSwap": [],
     "match": [],
     "liquidityDeposit": [],
     "liquidityWithdrawal": [],
     "assetCreation": [],
     "cancelation": []
    }
   },
   {
    "height": 6,
    "confirmations": 20,
    "body": {
     "reward": {
      "transaction": {
       "data": {
        "toAddress": "0000000000000000000000000000000000000000de47c9b2",
        "amount": {
         "str": "3.00000000",
         "E8": 300000000
        }
       },
       "hash": "a0a7a6bc04c7f3be940b464225eabb309273bd9de738045dc56ce10d6a49807f"
      },
      "historyId": 8
     },
     "wartTransfer": [],
     "tokenTransfer": [],
     "limitSwap": [],
     "match": [],
     "liquidityDeposit": [],
     "liquidityWithdrawal": [],
     "assetCreation": [],
     "cancelation": []
    }
   },
   {
    "height": 7,
    "confirmations": 19,
    "body": {
     "reward": {
      "transaction": {
       "data": {
        "toAddress": "0000000000000000000000000000000000000000de47c9b2",
        "amount": {
         "str": "3.00009992",
         "E8": 300009992
        }
       },
       "hash": "3d4786acec1659164abf635a2a1cf5847b55705533620e75dbf5d5c3ed9993ad"
      },
      "historyId": 9
     },
     "wartTransfer": [],
     "tokenTransfer": [],
     "limitSwap": [],
     "match": [],
     "liquidityDeposit": [],
     "liquidityWithdrawal": [],
     "assetCreation": [
      {
       "transaction": {
        "data": {
         "name": "TOK2",
         "supply": {
          "str": "100000000.0000",
          "u64": 1000000000000,
          "decimals": 4
         }
        },
        "processed": {
         "assetId": 7
        },
        "hash": "0e4825efffa294610d2ac376713e3bcc9b53d378e823834b64e5df01f75d3b0c",
        "signedCommon": {
         "originId": 1,
         "originAddress": "3661579d61abde5837a8686dc4d65348a2fc61b1",
         "fee": {
          "str": "0.00009992",
          "E8": 9992
         },
         "nonceId": 13,
         "pinHeight": 0
        }
       },
       "historyId": 10
      }
     ],
     "cancelation": []
    }
   },
   {
    "height": 8,
    "confirmations": 18,
    "body": {
     "reward": {
      "transaction": {
       "data": {
        "toAddress": "0000000000000000000000000000000000000000de47c9b2",
        "amount": {
         "str": "3.00009992",
         "E8": 300009992
        }
       },
       "hash": "576da5fde183f4bb7d3731cef903c8a01083875f0d40063235267580a4ea79ea"
      },
      "historyId": 11
     },
     "wartTransfer": [],
     "tokenTransfer": [],
     "limitSwap": [],
     "match": [],
     "liquidityDeposit": [
      {
       "transaction": {
        "data": {
         "baseAsset": {
          "hash": "f45b113119c7f7c000234f1090d5d181ab60b8b24526f1edd2f563aa1ca329f2",
          "id": 2,
          "name": "TOK2",
          "decimals": 4
         },
         "deposited": {
          "base": {
           "str": "99.9912",
           "u64": 999912,
           "decimals": 4
          },
          "quote": {
           "str": "0.00999912",
           "E8": 999912
          }
         }
        },
        "processed": {
         "sharesReceived": {
          "str": "99.9912",
          "u64": 999912,
          "decimals": 4
         }
        },
        "hash": "a3f6d1d3ee13cb8a85afb4dc28a3cbdb889c5c42ff1cc479469c4ff62fbca636",
        "signedCommon": {
         "originId": 1,
         "originAddress": "3661579d61abde5837a8686dc4d65348a2fc61b1",
         "fee": {
          "str": "0.00009992",
          "E8": 9992
         },
         "nonceId": 25,
         "pinHeight": 0
        }
       },
       "historyId": 12
      }
     ],
     "liquidityWithdrawal": [],
     "assetCreation": [],
     "cancelation": []
    }
   },
   {
    "height": 9,
    "confirmations": 17,
    "body": {
     "reward": {
      "transaction": {
       "data": {
        "toAddress": "2de77d5e23dc63e4c4149d394c979361e9e8e966336c8cd4",
        "amount": {
         "str": "3.00000000",
         "E8": 300000000
        }
       },
       "hash": "fed36c8133b30251421ac98948643d929490ab4d7ebbc934c98b4041391ad6d8"
      },
      "historyId": 13
     },
     "wartTransfer": [],
     "tokenTransfer": [],
     "limitSwap": [],
     "match": [],
     "liquidityDeposit": [],
     "liquidityWithdrawal": [],
     "assetCreation": [],
     "cancelation": []
    }
   },
   {
    "height": 10,
    "confirmations": 16,
    "body": {
     "reward": {
      "transaction": {
       "data": {
        "toAddress": "0000000000000000000000000000000000000000de47c9b2",
        "amount": {
         "str": "3.00000001",
         "E8": 300000001
        }
       },
       "hash": "e627d65d0bc43c6f4511e6fa802d886148ad7b431e5817ff757d599cb2a2fd84"
      },
      "historyId": 14
     },
     "wartTransfer": [],
     "tokenTransfer": [],
     "limitSwap": [
      {
       "transaction": {
        "data": {
         "baseAsset": {
          "hash": "0e4825efffa294610d2ac376713e3bcc9b53d378e823834b64e5df01f75d3b0c",
          "id": 7,
          "name": "TOK2",
          "decimals": 4
         },
         "amount": {
          "str": "0.00010000",
          "u64": 10000,
          "decimals": 8
         },
         "limit": {
          "precExponent10": 4,
          "exponent2": -2,
          "mantissa": 40000,
          "hex": "9c404d",
          "doubleAdjusted": 1,
          "doubleRaw": 10000
         },
         "buy": true
        },
        "processed": {
         "remaining": {
          "str": "0",
          "u64": 0,
          "decimals": 4
         }
        },
        "hash": "ed0ac4049643f1b8905b9b8ca209106869d47dbc677a679ef051b02eba21fa12",
        "signedCommon": {
         "originId": 9,
         "originAddress": "2de77d5e23dc63e4c4149d394c979361e9e8e966",
         "fee": {
          "str": "0.00000001",
          "E8": 1
         },
         "nonceId": 111,
         "pinHeight": 0
        }
       },
       "historyId": 15
      }
     ],
     "match": [
      {
       "transaction": {
        "data": {
         "baseAsset": {
          "hash": "0e4825efffa294610d2ac376713e3bcc9b53d378e823834b64e5df01f75d3b0c",
          "id": 7,
          "name": "TOK2",
          "decimals": 4
         },
         "poolBefore": {
          "base": {
           "str": "0",
           "u64": 0,
           "decimals": 4
          },
          "quote": {
           "str": "0",
           "E8": 0
          }
         },
         "poolAfter": {
          "base": {
           "str": "0",
           "u64": 0,
           "decimals": 4
          },
          "quote": {
           "str": "0",
           "E8": 0
          }
         },
         "buySwaps": [],
         "sellSwaps": []
        },
        "hash": "9716a483ea3d7a5d959697f201958149efb7ec851695c47c6d522aff3e0e0201"
       },
       "historyId": 16
      }
     ],
     "liquidityDeposit": [],
     "liquidityWithdrawal": [],
     "assetCreation": [],
     "cancelation": []
    }
   },
   {
    "height": 11,
    "confirmations": 15,
    "body": {
     "reward": {
      "transaction": {
       "data": {
        "toAddress": "0000000000000000000000000000000000000000de47c9b2",
        "amount": {
         "str": "3.00000001",
         "E8": 300000001
        }
       },
       "hash": "50f860f2c626c002aaac9db9b1db566ba280b10fc24fa470728b0b4ed88008f9"
      },
      "historyId": 17
     },
     "wartTransfer": [],
     "tokenTransfer": [],
     "limitSwap": [
      {
       "transaction": {
        "data": {
         "baseAsset": {
          "hash": "0e4825efffa294610d2ac376713e3bcc9b53d378e823834b64e5df01f75d3b0c",
          "id": 7,
          "name": "TOK2",
          "decimals": 4
         },
         "amount": {
          "str": "10000.0000",
          "u64": 100000000,
          "decimals": 4
         },
         "limit": {
          "precExponent10": 4,
          "exponent2": -2,
          "mantissa": 40000,
          "hex": "9c404d",
          "doubleAdjusted": 1,
          "doubleRaw": 10000
         },
         "buy": false
        },
        "processed": {
         "remaining": {
          "str": "10000.0000",
          "u64": 100000000,
          "decimals": 4
         }
        },
        "hash": "c37cb4d58fdc54e281c172ab50cc9cb30fefddcfc8092d8793a52161a75b9dda",
        "signedCommon": {
         "originId": 9,
         "originAddress": "2de77d5e23dc63e4c4149d394c979361e9e8e966",
         "fee": {
          "str": "0.00000001",
          "E8": 1
         },
         "nonceId": 1,
         "pinHeight": 0
        }
       },
       "historyId": 18
      }
     ],
     "match": [
      {
       "transaction": {
        "data": {
         "baseAsset": {
          "hash": "0e4825efffa294610d2ac376713e3bcc9b53d378e823834b64e5df01f75d3b0c",
          "id": 7,
          "name": "TOK2",
          "decimals": 4
         },
         "poolBefore": {
          "base": {
           "str": "0",
           "u64": 0,
           "decimals": 4
          },
          "quote": {
           "str": "0",
           "E8": 0
          }
         },
         "poolAfter": {
          "base": {
           "str": "0",
           "u64": 0,
           "decimals": 4
          },
          "quote": {
           "str": "0",
           "E8": 0
          }
         },
         "buySwaps": [],
         "sellSwaps": []
        },
        "hash": "906a6eb271ef650b695cd20931d7a4b6cecd550528a0c9277b3dc1431f822ae5"
       },
       "historyId": 19
      }
     ],
     "liquidityDeposit": [],
     "liquidityWithdrawal": [],
     "assetCreation": [],
     "cancelation": []
    }
   },
   {
    "height": 12,
    "confirmations": 14,
    "body": {
     "reward": {
      "transaction": {
       "data": {
        "toAddress": "0000000000000000000000000000000000000000de47c9b2",
        "amount": {
         "str": "3.99942401",
         "E8": 399942401
        }
       },
       "hash": "01c993d3b5112f8b29552892c6002a46f00cd860cca65e886123b77528c14196"
      },
      "historyId": 20
     },
     "wartTransfer": [],
     "tokenTransfer": [],
     "limitSwap": [
      {
       "transaction": {
        "data": {
         "baseAsset": {
          "hash": "0e4825efffa294610d2ac376713e3bcc9b53d378e823834b64e5df01f75d3b0c",
          "id": 7,
          "name": "TOK2",
          "decimals": 4
         },
         "amount": {
          "str": "1.00000000",
          "u64": 100000000,
          "decimals": 8
         },
         "limit": {
          "precExponent10": 4,
          "exponent2": -1,
          "mantissa": 40000,
          "hex": "9c404e",
          "doubleAdjusted": 2,
          "doubleRaw": 20000
         },
         "buy": true
        },
        "processed": {
         "remaining": {
          "str": "0",
          "u64": 0,
          "decimals": 4
         }
        },
        "hash": "dfbd7e086eee4cd230c5d19fe6ed03bce7f3e9f388c52b0f25a5192e4e380229",
        "signedCommon": {
         "originId": 9,
         "originAddress": "2de77d5e23dc63e4c4149d394c979361e9e8e966",
         "fee": {
          "str": "0.00000001",
          "E8": 1
         },
         "nonceId": 3,
         "pinHeight": 0
        }
       },
       "historyId": 22
      },
      {
       "transaction": {
        "data": {
         "baseAsset": {
          "hash": "0e4825efffa294610d2ac376713e3bcc9b53d378e823834b64e5df01f75d3b0c",
          "id": 7,
          "name": "TOK2",
          "decimals": 4
         },
         "amount": {
          "str": "300.0000",
          "u64": 3000000,
          "decimals": 4
         },
         "limit": {
          "precExponent10": 4,
          "exponent2": -5,
          "mantissa": 64000,
          "hex": "fa004a",
          "doubleAdjusted": 0.2,
          "doubleRaw": 2000
         },
         "buy": false
        },
        "processed": {
         "remaining": {
          "str": "300.0000",
          "u64": 3000000,
          "decimals": 4
         }
        },
        "hash": "c116706c3a6682c5e3e358fbae00c0b1aab0a8e3cff1b5883ab037bcf567b9b8",
        "signedCommon": {
         "originId": 9,
         "originAddress": "2de77d5e23dc63e4c4149d394c979361e9e8e966",
         "fee": {
          "str": "0.99942400",
          "E8": 99942400
         },
         "nonceId": 12213,
         "pinHeight": 0
        }
       },
       "historyId": 21
      }
     ],
     "match": [
      {
       "transaction": {
        "data": {
         "baseAsset": {
          "hash": "0e4825efffa294610d2ac376713e3bcc9b53d378e823834b64e5df01f75d3b0c",
          "id": 7,
          "name": "TOK2",
          "decimals": 4
         },
         "poolBefore": {
          "base": {
           "str": "0",
           "u64": 0,
           "decimals": 4
          },
          "quote": {
           "str": "0",
           "E8": 0
          }
         },
         "poolAfter": {
          "base": {
           "str": "0",
           "u64": 0,
           "decimals": 4
          },
          "quote": {
           "str": "0",
           "E8": 0
          }
         },
         "buySwaps": [],
         "sellSwaps": []
        },
        "hash": "dd149af828af5edd0c8c1accb0ff0ff400082d65207ec616fdb7b858b159aa87"
       },
       "historyId": 23
      }
     ],
     "liquidityDeposit": [],
     "liquidityWithdrawal": [],
     "assetCreation": [],
     "cancelation": []
    }
   },
   {
    "height": 13,
    "confirmations": 13,
    "body": {
     "reward": {
      "transaction": {
       "data": {
        "toAddress": "0000000000000000000000000000000000000000de47c9b2",
        "amount": {
         "str": "3.00000002",
         "E8": 300000002
        }
       },
       "hash": "6ff011029afa60268b580f5881498c99f2fd6e8e6d5844c1417943bdcb2d224a"
      },
      "historyId": 24
     },
     "wartTransfer": [],
     "tokenTransfer": [],
     "limitSwap": [
      {
       "transaction": {
        "data": {
         "baseAsset": {
          "hash": "0e4825efffa294610d2ac376713e3bcc9b53d378e823834b64e5df01f75d3b0c",
          "id": 7,
          "name": "TOK2",
          "decimals": 4
         },
         "amount": {
          "str": "1.00000000",
          "u64": 100000000,
          "decimals": 8
         },
         "limit": {
          "precExponent10": 4,
          "exponent2": 4,
          "mantissa": 62500,
          "hex": "f42453",
          "doubleAdjusted": 100,
          "doubleRaw": 1000000
         },
         "buy": true
        },
        "processed": {
         "remaining": {
          "str": "0",
          "u64": 0,
          "decimals": 4
         }
        },
        "hash": "8476384a00e0b5815c08ed28fa8142395748fb882b399a15a61b1d75472d770c",
        "signedCommon": {
         "originId": 9,
         "originAddress": "2de77d5e23dc63e4c4149d394c979361e9e8e966",
         "fee": {
          "str": "0.00000001",
          "E8": 1
         },
         "nonceId": 123213,
         "pinHeight": 0
        }
       },
       "historyId": 26
      },
      {
       "transaction": {
        "data": {
         "baseAsset": {
          "hash": "0e4825efffa294610d2ac376713e3bcc9b53d378e823834b64e5df01f75d3b0c",
          "id": 7,
          "name": "TOK2",
          "decimals": 4
         },
         "amount": {
          "str": "1000.0000",
          "u64": 10000000,
          "decimals": 4
         },
         "limit": {
          "precExponent10": 4,
          "exponent2": -6,
          "mantissa": 64000,
          "hex": "fa0049",
          "doubleAdjusted": 0.1,
          "doubleRaw": 1000
         },
         "buy": false
        },
        "processed": {
         "remaining": {
          "str": "979.9990",
          "u64": 9799990,
          "decimals": 4
         }
        },
        "hash": "37bb2e782a32065a61962ae5e9637bfb8f45b5874db64a5b4bbd54c115d0a183",
        "signedCommon": {
         "originId": 9,
         "originAddress": "2de77d5e23dc63e4c4149d394c979361e9e8e966",
         "fee": {
          "str": "0.00000001",
          "E8": 1
         },
         "nonceId": 32423,
         "pinHeight": 0
        }
       },
       "historyId": 25
      }
     ],
     "match": [
      {
       "transaction": {
        "data": {
         "baseAsset": {
          "hash": "0e4825efffa294610d2ac376713e3bcc9b53d378e823834b64e5df01f75d3b0c",
          "id": 7,
          "name": "TOK2",
          "decimals": 4
         },
         "poolBefore": {
          "base": {
           "str": "0",
           "u64": 0,
           "decimals": 4
          },
          "quote": {
           "str": "0",
           "E8": 0
          }
         },
         "poolAfter": {
          "base": {
           "str": "0",
           "u64": 0,
           "decimals": 4
          },
          "quote": {
           "str": "0",
           "E8": 0
          }
         },
         "buySwaps": [],
         "sellSwaps": []
        },
        "hash": "3d02070a3b689bce949a39d33a67051c88384c5b6002081a40bcd6e716de6d60"
       },
       "historyId": 27
      }
     ],
     "liquidityDeposit": [],
     "liquidityWithdrawal": [],
     "assetCreation": [],
     "cancelation": []
    }
   },
   {
    "height": 14,
    "confirmations": 12,
    "body": {
     "reward": {
      "transaction": {
       "data": {
        "toAddress": "0000000000000000000000000000000000000000de47c9b2",
        "amount": {
         "str": "3.00000001",
         "E8": 300000001
        }
       },
       "hash": "ef27ccf46e2e4233c8e7a4f3869b2c74f925821561df28295cd98e6ffeff7c8e"
      },
      "historyId": 28
     },
     "wartTransfer": [],
     "tokenTransfer": [],
     "limitSwap": [
      {
       "transaction": {
        "data": {
         "baseAsset": {
          "hash": "0e4825efffa294610d2ac376713e3bcc9b53d378e823834b64e5df01f75d3b0c",
          "id": 7,
          "name": "TOK2",
          "decimals": 4
         },
         "amount": {
          "str": "1000.0000",
          "u64": 10000000,
          "decimals": 4
         },
         "limit": {
          "precExponent10": 4,
          "exponent2": -6,
          "mantissa": 64000,
          "hex": "fa0049",
          "doubleAdjusted": 0.1,
          "doubleRaw": 1000
         },
         "buy": false
        },
        "processed": {
         "remaining": {
          "str": "0",
          "u64": 0,
          "decimals": 4
         }
        },
        "hash": "91f4fc7cdb5d3aaaf9eecdee8023b7186b594677450b2f2f300877a6bfdbf5f4",
        "signedCommon": {
         "originId": 9,
         "originAddress": "2de77d5e23dc63e4c4149d394c979361e9e8e966",
         "fee": {
          "str": "0.00000001",
          "E8": 1
         },
         "nonceId": 9,
         "pinHeight": 0
        }
       },
       "historyId": 29
      }
     ],
     "match": [
      {
       "transaction": {
        "data": {
         "baseAsset": {
          "hash": "0e4825efffa294610d2ac376713e3bcc9b53d378e823834b64e5df01f75d3b0c",
          "id": 7,
          "name": "TOK2",
          "decimals": 4
         },
         "poolBefore": {
          "base": {
           "str": "0",
           "u64": 0,
           "decimals": 4
          },
          "quote": {
           "str": "0",
           "E8": 0
          }
         },
         "poolAfter": {
          "base": {
           "str": "0",
           "u64": 0,
           "decimals": 4
          },
          "quote": {
           "str": "0",
           "E8": 0
          }
         },
         "buySwaps": [],
         "sellSwaps": []
        },
        "hash": "887dfb7c44c4a2e88b4ad1fb245166e475a3a38c895c26163aa725704c12028c"
       },
       "historyId": 30
      }
     ],
     "liquidityDeposit": [],
     "liquidityWithdrawal": [],
     "assetCreation": [],
     "cancelation": []
    }
   },
   {
    "height": 15,
    "confirmations": 11,
    "body": {
     "reward": {
      "transaction": {
       "data": {
        "toAddress": "0000000000000000000000000000000000000000de47c9b2",
        "amount": {
         "str": "3.00000000",
         "E8": 300000000
        }
       },
       "hash": "43c3f3132ccbcef167b42c1632cdcef02fa45f8dc1ca274d22a0a1f0336f4824"
      },
      "historyId": 31
     },
     "wartTransfer": [],
     "tokenTransfer": [],
     "limitSwap": [],
     "match": [],
     "liquidityDeposit": [],
     "liquidityWithdrawal": [],
     "assetCreation": [],
     "cancelation": []
    }
   },
   {
    "height": 16,
    "confirmations": 10,
    "body": {
     "reward": {
      "transaction": {
       "data": {
        "toAddress": "0000000000000000000000000000000000000000de47c9b2",
        "amount": {
         "str": "3.00000000",
         "E8": 300000000
        }
       },
       "hash": "1212214fc805cd150607aa4db0b442afa7a78484d92a62974c61b1ec51a3608c"
      },
      "historyId": 32
     },
     "wartTransfer": [],
     "tokenTransfer": [],
     "limitSwap": [],
     "match": [],
     "liquidityDeposit": [],
     "liquidityWithdrawal": [],
     "assetCreation": [],
     "cancelation": []
    }
   },
   {
    "height": 17,
    "confirmations": 9,
    "body": {
     "reward": {
      "transaction": {
       "data": {
        "toAddress": "0000000000000000000000000000000000000000de47c9b2",
        "amount": {
         "str": "3.00000000",
         "E8": 300000000
        }
       },
       "hash": "ac54374e7fa0c461466c51dd23304bf5222dba1c89ab2b8fd9d486d7c62d977b"
      },
      "historyId": 33
     },
     "wartTransfer": [],
     "tokenTransfer": [],
     "limitSwap": [],
     "match": [],
     "liquidityDeposit": [],
     "liquidityWithdrawal": [],
     "assetCreation": [],
     "cancelation": []
    }
   },
   {
    "height": 18,
    "confirmations": 8,
    "body": {
     "reward": {
      "transaction": {
       "data": {
        "toAddress": "0000000000000000000000000000000000000000de47c9b2",
        "amount": {
         "str": "3.00000000",
         "E8": 300000000
        }
       },
       "hash": "bca37d6b73ad8d73d1185f528e92b18e62f8f42023a62965b2cadf695af556e7"
      },
      "historyId": 34
     },
     "wartTransfer": [],
     "tokenTransfer": [],
     "limitSwap": [],
     "match": [],
     "liquidityDeposit": [],
     "liquidityWithdrawal": [],
     "assetCreation": [],
     "cancelation": []
    }
   },
   {
    "height": 19,
    "confirmations": 7,
    "body": {
     "reward": {
      "transaction": {
       "data": {
        "toAddress": "0000000000000000000000000000000000000000de47c9b2",
        "amount": {
         "str": "3.00000000",
         "E8": 300000000
        }
       },
       "hash": "a67d70cc28acba01c93798891bbf2cd390ee8a2ea45cc446b730380a02b0b145"
      },
      "historyId": 35
     },
     "wartTransfer": [],
     "tokenTransfer": [],
     "limitSwap": [],
     "match": [],
     "liquidityDeposit": [],
     "liquidityWithdrawal": [],
     "assetCreation": [],
     "cancelation": []
    }
   },
   {
    "height": 20,
    "confirmations": 6,
    "body": {
     "reward": {
      "transaction": {
       "data": {
        "toAddress": "0000000000000000000000000000000000000000de47c9b2",
        "amount": {
         "str": "3.00000001",
         "E8": 300000001
        }
       },
       "hash": "7fcb1490e61206bcca62ce8694d58f6e6ec3927cd6b06ab36f562e628ae484d4"
      },
      "historyId": 36
     },
     "wartTransfer": [],
     "tokenTransfer": [],
     "limitSwap": [
      {
       "transaction": {
        "data": {
         "baseAsset": {
          "hash": "0e4825efffa294610d2ac376713e3bcc9b53d378e823834b64e5df01f75d3b0c",
          "id": 7,
          "name": "TOK2",
          "decimals": 4
         },
         "amount": {
          "str": "100.0000",
          "u64": 1000000,
          "decimals": 4
         },
         "limit": {
          "precExponent10": 4,
          "exponent2": -6,
          "mantissa": 64000,
          "hex": "fa0049",
          "doubleAdjusted": 0.1,
          "doubleRaw": 1000
         },
         "buy": false
        },
        "processed": {
         "remaining": {
          "str": "100.0000",
          "u64": 1000000,
          "decimals": 4
         }
        },
        "hash": "c6fbade2df322654ece3bd7cb372a80812630a2478eff8db801bd89062d8f209",
        "signedCommon": {
         "originId": 9,
         "originAddress": "2de77d5e23dc63e4c4149d394c979361e9e8e966",
         "fee": {
          "str": "0.00000001",
          "E8": 1
         },
         "nonceId": 99,
         "pinHeight": 0
        }
       },
       "historyId": 37
      }
     ],
     "match": [
      {
       "transaction": {
        "data": {
         "baseAsset": {
          "hash": "0e4825efffa294610d2ac376713e3bcc9b53d378e823834b64e5df01f75d3b0c",
          "id": 7,
          "name": "TOK2",
          "decimals": 4
         },
         "poolBefore": {
          "base": {
           "str": "0",
           "u64": 0,
           "decimals": 4
          },
          "quote": {
           "str": "0",
           "E8": 0
          }
         },
         "poolAfter": {
          "base": {
           "str": "0",
           "u64": 0,
           "decimals": 4
          },
          "quote": {
           "str": "0",
           "E8": 0
          }
         },
         "buySwaps": [
          {
           "swapped": {
            "base": {
             "str": "10.0000",
             "u64": 100000,
             "decimals": 4
            },
            "quote": {
             "str": "1.00000000",
             "E8": 100000000
            }
           },
           "historyId": 26
          },
          {
           "swapped": {
            "base": {
             "str": "10.0000",
             "u64": 100000,
             "decimals": 4
            },
            "quote": {
             "str": "1.00000000",
             "E8": 100000000
            }
           },
           "historyId": 22
          },
          {
           "swapped": {
            "base": {
             "str": "0.0010",
             "u64": 10,
             "decimals": 4
            },
            "quote": {
             "str": "0.00010000",
             "E8": 10000
            }
           },
           "historyId": 15
          }
         ],
         "sellSwaps": [
          {
           "swapped": {
            "base": {
             "str": "20.0010",
             "u64": 200010,
             "decimals": 4
            },
            "quote": {
             "str": "2.00010000",
             "E8": 200010000
            }
           },
           "historyId": 25
          }
         ]
        },
        "hash": "2ecefa2fb405cd4ab51c5a392ee38d1f26d51885fb1e9cac059e2bc059b53949"
       },
       "historyId": 38
      }
     ],
     "liquidityDeposit": [],
     "liquidityWithdrawal": [],
     "assetCreation": [],
     "cancelation": []
    }
   },
   {
    "height": 21,
    "confirmations": 5,
    "body": {
     "reward": {
      "transaction": {
       "data": {
        "toAddress": "0000000000000000000000000000000000000000de47c9b2",
        "amount": {
         "str": "3.00039969",
         "E8": 300039969
        }
       },
       "hash": "91b5330d1a3f3c9495bda05a46a0dcae86a6af441493c0a68817872d56a5def8"
      },
      "historyId": 39
     },
     "wartTransfer": [
      {
       "transaction": {
        "data": {
         "toAddress": "0000000000000000000000000000000000000000de47c9b2",
         "amount": {
          "str": "1.00000000",
          "E8": 100000000
         }
        },
        "hash": "897b3cc72950b41ee71527dc05a5bb6745801abbfb6179ad91dd0ccf2834eea8",
        "signedCommon": {
         "originId": 1,
         "originAddress": "3661579d61abde5837a8686dc4d65348a2fc61b1",
         "fee": {
          "str": "0.00009992",
          "E8": 9992
         },
         "nonceId": 0,
         "pinHeight": 0
        }
       },
       "historyId": 40
      }
     ],
     "tokenTransfer": [
      {
       "transaction": {
        "data": {
         "toAddress": "0000000000000000000000000000000000000000de47c9b2",
         "amount": {
          "str": "99.9912",
          "u64": 999912,
          "decimals": 4
         },
         "asset": {
          "hash": "f45b113119c7f7c000234f1090d5d181ab60b8b24526f1edd2f563aa1ca329f2",
          "id": 2,
          "name": "TOK2",
          "decimals": 4
         },
         "isLiquidity": false,
         "tokenSpec": "asset:f45b113119c7f7c000234f1090d5d181ab60b8b24526f1edd2f563aa1ca329f2"
        },
        "hash": "87a8f53490f32a29905554f4e23699774dd8c3e38c34033ba4812720824b5404",
        "signedCommon": {
         "originId": 1,
         "originAddress": "3661579d61abde5837a8686dc4d65348a2fc61b1",
         "fee": {
          "str": "0.00009992",
          "E8": 9992
         },
         "nonceId": 1,
         "pinHeight": 0
        }
       },
       "historyId": 41
      }
     ],
     "limitSwap": [],
     "match": [],
     "liquidityDeposit": [
      {
       "transaction": {
        "data": {
         "baseAsset": {
          "hash": "0e4825efffa294610d2ac376713e3bcc9b53d378e823834b64e5df01f75d3b0c",
          "id": 7,
          "name": "TOK2",
          "decimals": 4
         },
         "deposited": {
          "base": {
           "str": "100.0000",
           "u64": 1000000,
           "decimals": 4
          },
          "quote": {
           "str": "1.00000000",
           "E8": 100000000
          }
         }
        },
        "processed": {
         "sharesReceived": {
          "str": "1000.0000",
          "u64": 10000000,
          "decimals": 4
         }
        },
        "hash": "1db23925133e7d397c8acec355bfe04a8186ba4fb5926e16fcbf74293b300508",
        "signedCommon": {
         "originId": 9,
         "originAddress": "2de77d5e23dc63e4c4149d394c979361e9e8e966",
         "fee": {
          "str": "0.00000001",
          "E8": 1
         },
         "nonceId": 111111,
         "pinHeight": 0
        }
       },
       "historyId": 43
      },
      {
       "transaction": {
        "data": {
         "baseAsset": {
          "hash": "f45b113119c7f7c000234f1090d5d181ab60b8b24526f1edd2f563aa1ca329f2",
          "id": 2,
          "name": "TOK2",
          "decimals": 4
         },
         "deposited": {
          "base": {
           "str": "99.9912",
           "u64": 999912,
           "decimals": 4
          },
          "quote": {
           "str": "0.00999912",
           "E8": 999912
          }
         }
        },
        "processed": {
         "sharesReceived": {
          "str": "99.9912",
          "u64": 999912,
          "decimals": 4
         }
        },
        "hash": "47c50348397eed92b918501ec13c368e11df533388d199c895aebf57d9d8d0d3",
        "signedCommon": {
         "originId": 1,
         "originAddress": "3661579d61abde5837a8686dc4d65348a2fc61b1",
         "fee": {
          "str": "0.00009992",
          "E8": 9992
         },
         "nonceId": 3,
         "pinHeight": 0
        }
       },
       "historyId": 42
      }
     ],
     "liquidityWithdrawal": [],
     "assetCreation": [
      {
       "transaction": {
        "data": {
         "name": "TOK2",
         "supply": {
          "str": "100000000.0000",
          "u64": 1000000000000,
          "decimals": 4
         }
        },
        "processed": {
         "assetId": 11
        },
        "hash": "d645efa5c0a64b409e44725d1d9b84ed7d800fca8ca5b3d71301d361652eddbc",
        "signedCommon": {
         "originId": 1,
         "originAddress": "3661579d61abde5837a8686dc4d65348a2fc61b1",
         "fee": {
          "str": "0.00009992",
          "E8": 9992
         },
         "nonceId": 2,
         "pinHeight": 0
        }
       },
       "historyId": 44
      }
     ],
     "cancelation": []
    }
   },
   {
    "height": 22,
    "confirmations": 4,
    "body": {
     "reward": {
      "transaction": {
       "data": {
        "toAddress": "0000000000000000000000000000000000000000de47c9b2",
        "amount": {
         "str": "3.00000000",
         "E8": 300000000
        }
       },
       "hash": "2c8201847cc4f49b956a909eb90ecb090d2053453819e519774f08f0ac5419b8"
      },
      "historyId": 45
     },
     "wartTransfer": [],
     "tokenTransfer": [],
     "limitSwap": [],
     "match": [],
     "liquidityDeposit": [],
     "liquidityWithdrawal": [],
     "assetCreation": [],
     "cancelation": []
    }
   },
   {
    "height": 23,
    "confirmations": 3,
    "body": {
     "reward": {
      "transaction": {
       "data": {
        "toAddress": "0000000000000000000000000000000000000000de47c9b2",
        "amount": {
         "str": "3.00000000",
         "E8": 300000000
        }
       },
       "hash": "b616e4ad61747aa42abe9a5a68bea168f946e897ce51b247c082d5b430f90d88"
      },
      "historyId": 46
     },
     "wartTransfer": [],
     "tokenTransfer": [],
     "limitSwap": [],
     "match": [],
     "liquidityDeposit": [],
     "liquidityWithdrawal": [],
     "assetCreation": [],
     "cancelation": []
    }
   },
   {
    "height": 24,
    "confirmations": 2,
    "body": {
     "reward": {
      "transaction": {
       "data": {
        "toAddress": "0000000000000000000000000000000000000000de47c9b2",
        "amount": {
         "str": "3.00000000",
         "E8": 300000000
        }
       },
       "hash": "78e50eb43f6e7ddc3ced5bc1b4375413ec4e87ba2afd63905c534d684a80e9e2"
      },
      "historyId": 47
     },
     "wartTransfer": [],
     "tokenTransfer": [],
     "limitSwap": [],
     "match": [],
     "liquidityDeposit": [],
     "liquidityWithdrawal": [],
     "assetCreation": [],
     "cancelation": []
    }
   },
   {
    "height": 25,
    "confirmations": 1,
    "body": {
     "reward": {
      "transaction": {
       "data": {
        "toAddress": "2de77d5e23dc63e4c4149d394c979361e9e8e966336c8cd4",
        "amount": {
         "str": "3.00000000",
         "E8": 300000000
        }
       },
       "hash": "a60e4e800b5461ff7371e84d812fdca1907ae3e58c411c6e85ab7e52b0e3d21f"
      },
      "historyId": 48
     },
     "wartTransfer": [],
     "tokenTransfer": [],
     "limitSwap": [],
     "match": [],
     "liquidityDeposit": [],
     "liquidityWithdrawal": [],
     "assetCreation": [],
     "cancelation": []
    }
   }
  ],
  "fromId": 1
 }
}
```
===

### `GET /transaction/minfee`

Show the minimum mempool fee required by this node. Transactions with a lower fee will not be accepted or requested by the node on mempool level. This does not affect block validation, i.e. transaction fees in mined blcoks are not matched against this value.

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

Set the minimum fee required fee for transactions to be accepted into mempool. The integer argument `:feeE8` is the WART value multiplied by 10^8. The number of transactions that were excluded from the mempool by applying the submitted minimal fee is returned in the `deleted` field of the return value.

Example output:
```json
{
 "code": 0,
 "data": {
  "deleted": 10
 }
}
```

### `GET /chain/head`

Show the current chain head and sync status.

Example:
```json
{
 "code": 0,
 "data": {
  "chainHead": {
   "difficulty": 14142941042437.885,
   "hash": "80b200a9ac37d7c41d98ad1cc2d6add8ab50a3905bf52d8347b56955923c28bc",
   "hashrate": 140000000000000,
   "height": 1949776,
   "is_janushash": true,
   "pinHash": "568e30a9abdbc510c9a21ec2dcb636e66cd4d1fb60028a31c65906069114e988",
   "pinHeight": 1949760,
   "worksum": 29360669698622464000,
   "worksumHex": "0x00000000000000000000000000000000000000000000000197760d8809f796a0"
  },
  "synced": true
 }
}
```
### `GET /chain/grid`

Show the header grid used for chain synchronization. The grid represents block headers at mesh size 8640. Returns a list of these block headers in hex format.

Example:
```json
{
 "code": 0,
 "data": {
  "headers": [
   "05b42d206f8a4b2b8094199fd28423151af8aa54a5073ac6e6805f42096dc3a20ae7cde7f3c41bf363fe94eb448bf924f791f77c22a2ebd6a3a5d8948f8aff019f072bea",
   "8c8811ee10a9c5c7d23f6f4791cbf03b781531574d53f224278cb78531a4c2fe0ae7cde7dcfeba982307da35c0098709c20f4ea0a4f04e0bbcd1b0c34c8ecc30157159eb"
  ]
 }
}
```

### `GET /chain/block/:height/hash`

Show the hash of block at specified height for positive `:height` parameter or genesis hash for `:height` parameter `0`.

Example:
```json
{
 "code": 0,
 "data": "80b200a9ac37d7c41d98ad1cc2d6add8ab50a3905bf52d8347b56955923c28bc"
}
```

### `GET /chain/block/:id/header`

Show the header of a specific block. The `:id` parameter can be a block height (positive integer) or block hash (hex).

==- Example:
```json
{
 "code": 0,
 "data": {
  "header": {
   "hash": "80b200a9ac37d7c41d98ad1cc2d6add8ab50a3905bf52d8347b56955923c28bc",
   "merkleroot": "f3c41bf363fe94eb448bf924f791f77c22a2ebd6a3a5d8948f8aff019f072bea",
   "nonce": "2719fef5",
   "pow": {
    "floatSha256t": 0.010330632328987122,
    "floatVerus": 6.267357060779177e-13,
    "hashSha256t": "02a507400db273886cd1bdc61e05bc4bde6978d74ce6e48d33a46fa705c59ae1",
    "hashVerus": "0000000000b069112cc0caf14507169f6adf1e5278a2ef286cddc89f1162d7ca",
    "verusV2.2": true
   },
   "prevHash": "05b42d206f8a4b2b8094199fd28423151af8aa54a5073ac6e6805f42096dc3a2",
   "raw": "05b42d206f8a4b2b8094199fd28423151af8aa54a5073ac6e6805f42096dc3a20ae7cde7f3c41bf363fe94eb448bf924f791f77c22a2ebd6a3a5d8948f8aff019f072bea0000000366f2fefb2719fef5",
   "target": "0ae7cde7",
   "time": 1727201019,
   "version": "00000003"
  },
  "height": 1949766
 }
}
```
===
### `GET /chain/block/:id/binary`

Show the raw binary data of a specific block. The `structure` field is a tree of byte-range annotations with `beginOffset` and `endOffset` positions and optional nested `children`.

==- Example:
```json
{
 "code": 0,
 "data": {
  "bytes": "0000000000000000000000013661579d61abde5837a8686dc4d65348a2fc61b100000000000000010000000011e1a30000000000000000000000",
  "structure": [
   {
    "tag": "extraNonce",
    "beginOffset": 0,
    "endOffset": 10,
    "children": []
   },
   {
    "tag": "newAddresses",
    "beginOffset": 10,
    "endOffset": 32,
    "children": [
     {
      "tag": "length",
      "beginOffset": 10,
      "endOffset": 12,
      "children": []
     }
    ]
   },
   {
    "tag": "reward",
    "beginOffset": 32,
    "endOffset": 48,
    "children": [
     {
      "tag": "toAccountId",
      "beginOffset": 32,
      "endOffset": 40,
      "children": []
     },
     {
      "tag": "wart",
      "beginOffset": 40,
      "endOffset": 48,
      "children": []
     }
    ]
   },
   {
    "tag": "wartTransfer",
    "beginOffset": 48,
    "endOffset": 52,
    "children": [
     {
      "tag": "length",
      "beginOffset": 48,
      "endOffset": 52,
      "children": []
     }
    ]
   },
   {
    "tag": "cancelation",
    "beginOffset": 52,
    "endOffset": 54,
    "children": [
     {
      "tag": "length",
      "beginOffset": 52,
      "endOffset": 54,
      "children": []
     }
    ]
   },
   {
    "tag": "tokenSections",
    "beginOffset": 54,
    "endOffset": 56,
    "children": [
     {
      "tag": "length",
      "beginOffset": 54,
      "endOffset": 56,
      "children": []
     }
    ]
   },
   {
    "tag": "assetCreation",
    "beginOffset": 56,
    "endOffset": 58,
    "children": [
     {
      "tag": "length",
      "beginOffset": 56,
      "endOffset": 58,
      "children": []
     }
    ]
   }
  ]
 }
}
```
===
### `GET /chain/block/:id`

Show the full block including header and body. The `:id` parameter can be a block height (integer) or block hash (hex). The `body` parameter has the same structure as returned in `perBlock` in the [`/transaction/latest`](#get-transactionlatest) endpoint except that the `reward` transaction cannot be `null` here. See [Transactions](./rest/transactions.md) for details on each `body` field.

==- Example:
```json
{
 "code": 0,
 "data": {
  "header": {
   "raw": "b29988793c29118a842ba38ea4ab13d43e3dce3f04511ba6ca3340542dfd8537077fffffeab3b88a101b745e8d7b530eb129a789f571a4718edc07435ae74998981a3a7f000000036948246d00000000",
   "time": {
    "UTC": "2025-12-21 16:46:37 UTC",
    "timestamp": 1766335597
   },
   "target": "077fffff",
   "hash": "6851377f19b814ab3a4354c811a96908afba9c47cf3e4b85048d95de11e3f6aa",
   "pow": {
    "verusV2_2": false,
    "hashVerus": "2443706cf33a161fd811fa69c7180787f4ddbc518f9e8cacd4efef29b3cc22a9",
    "hashSha256t": "353389ceb7bccd1efe58d3ff45b97c4efd99d0a460070d554a2f78edd92482b7",
    "floatVerus": 0.14165403973311186,
    "floatSha256t": 0.20781766204163432
   },
   "merkleroot": "eab3b88a101b745e8d7b530eb129a789f571a4718edc07435ae74998981a3a7f",
   "nonce": "00000000",
   "prevHash": "b29988793c29118a842ba38ea4ab13d43e3dce3f04511ba6ca3340542dfd8537",
   "version": "00000003"
  },
  "body": {
   "reward": {
    "transaction": {
     "data": {
      "toAddress": "3661579d61abde5837a8686dc4d65348a2fc61b1fe5f4093",
      "amount": {
       "str": "3.00000000",
       "E8": 300000000
      }
     },
     "hash": "da3dcfbc0dcb60be544a8510589f973b82651b48ddcb7943718dc3dcf93ae234"
    },
    "historyId": 1
   },
   "wartTransfer": [],
   "tokenTransfer": [],
   "limitSwap": [],
   "match": [],
   "liquidityDeposit": [],
   "liquidityWithdrawal": [],
   "assetCreation": [],
   "cancelation": []
  },
  "confirmations": 25,
  "height": 1
 }
}
```
===
### `GET /chain/mine/:account`

Generate the block header, body, and merkle prefix data required for mining. The `:account` parameter is the miner's account address (hex).

==- Example:
```json
{
 "code": 0,
 "data": {
  "blockReward": { "E8": 300000000, "str": "3.00000000" },
  "body": "0a3661579d2a59c6a98042f29ec87d1fc0f44a6c3ceea2479766d6e8f5a",
  "difficulty": 14142941042437.885,
  "header": "05b42d206f8a4b2b8094199fd28423151af8aa54a5073ac6e6805f42096dc3a2",
  "height": 1949776,
  "merklePrefix": "f3c41bf363fe94eb448bf924f791f77c22a2ebd6a3a5d8948f8aff019f072bea",
  "synced": true,
  "testnet": false
 }
}
```
===
### `GET /chain/txcache`

Show the transaction cache, a list of transaction ids which have been recently mined. This is Warthog's mechanism to prevent double spend. Only most recent transaction id's need to be cached because every transaction signature includes the block hash of a previous block (`pinHash`) which cannot be too far in the past. Only transaction id's of past transactions that are reachable by this back-reference are required to be cached to avoid double-spend.
==- Example:
```json
{
 "code": 0,
 "data": [
  { "accountId": 2, "nonceId": 3, "pinHeight": 10 },
  { "accountId": 5, "nonceId": 1, "pinHeight": 0 }
 ]
}
```
===
### `GET /chain/hashrate/:window`

Show the estimated hashrate based on the latest N blocks. The `:window` parameter is the number of blocks to average over. If `:window` is greater than the chain length, only the available blocks are used to form the average.

==- Example:
```json
{
 "code": 0,
 "data": {
  "estimate": 140000000000000,
  "nBlocks": 120
 }
}
```
===
### `POST /chain/append`

Append a mined block to the chain. Takes a block payload in the request body. Returns `null` on success.

==- Example:
```json
{
 "code": 0,
 "data": null
}
```
===
### `GET /asset/complete?namePrefix=...&hashPrefix=...`

Search assets by name and/or hash prefix. Both query parameters are optional; if no parameter is provided, a truncated list of all assets is returned.

==- Example:
```json
{
 "code": 0,
 "data": {
  "hashPrefix": "f45b",
  "match": [
   {
    "decimals": 4,
    "groupId": 0,
    "hash": "f45b1131a2b3c4d5e6f7a8b9c0d1e2f3a4b5c6d7e8f9a0b1c2d3e4f5a6b",
    "height": 12345,
    "id": 2,
    "name": "TOK2",
    "ownerAccountId": 1,
    "parentId": null,
    "totalSupply": { "decimals": 4, "str": "100000000.0000", "u64": 1000000000000 }
   }
  ],
  "namePrefix": "TO"
 }
}
```
===
### `GET /asset/lookup/:asset`

Asset lookup by ID or hash. The `:asset` parameter can be the asset ID (integer) or the asset hash (hex string).

==- Example:
```json
{
 "code": 0,
 "data": {
  "decimals": 4,
  "groupId": 0,
  "hash": "f45b1131a2b3c4d5e6f7a8b9c0d1e2f3a4b5c6d7e8f9a0b1c2d3e4f5a6b",
  "height": 12345,
  "id": 2,
  "name": "TOK2",
  "ownerAccountId": 1,
  "parentId": null,
  "totalSupply": { "decimals": 4, "str": "100000000.0000", "u64": 1000000000000 }
 }
}
```
===

### `GET /dex/market/:asset`

Show orders, liquidity pool and matching for the market with specified base asset and WART as quote asset. The `:asset` parameter can be the asset hash or the asset ID. The returned structure is as follows:

```json
{
 "assetToWartSwaps": [
  {
   "amount": { "str": "100.0000", "u64": 1000000, "decimals": 4 },
   "filled": { "str": "0", "u64": 0, "decimals": 4 },
   "inMempool": false,
   "limit": { "doubleAdjusted": 0.1, "doubleRaw": 1000.0, "exponent2": -6, "hex": "fa0049", "mantissa": 64000, "precExponent10": 4 },
   "txHash": "c6fbade2..."
  }
 ],
 "baseAsset": { "decimals": 4, "hash": "0e4825ef...", "id": 7, "name": "TOK2" },
 "liquidityPool": {
  "asset": { "str": "1000.0000", "u64": 10000000, "decimals": 4 },
  "shares": { "str": "100.0000", "u64": 1000000, "decimals": 4 },
  "wart": { "E8": 100000000, "str": "1.00000000" }
 },
 "match": {
  "filled": { "base": { "str": "100.0000", "u64": 1000000, "decimals": 4 }, "quote": { "E8": 100000000, "str": "1.00000000" }},
  "toPool": { "amount": { "str": "100.0000", "u64": 1000000, "decimals": 4 }, "isQuote": false},
 }
 "wartToAssetSwaps": [...], // same structure as "wartToAssetSwaps"
}
```
The order book is represented by `assetToWartSwaps` and `wartToAssetSwaps` fields which are both arrays containing elements of the form

```json
{
 "amount": { "str": "100.0000", "u64": 1000000, "decimals": 4 },
 "filled": { "str": "0", "u64": 0, "decimals": 4 },
 "inMempool": false,
 "limit": { "doubleAdjusted": 0.1, "doubleRaw": 1000.0, "exponent2": -6, "hex": "fa0049", "mantissa": 64000, "precExponent10": 4 },
 "txHash": "c6fbade2..."
}
```

The `match` field uses Warthog's custom sandwich-free [matching engine for sandwich-free matching](/unique-features/sandwich-proof-defi/matching-engine).

==- Example output of `/dex/market/0e4825efffa294610d2ac376713e3bcc9b53d378e823834b64e5df01f75d3b0c`:
```json
{
 "code": 0,
 "data": {
  "baseAsset": {
   "hash": "0e4825efffa294610d2ac376713e3bcc9b53d378e823834b64e5df01f75d3b0c",
   "id": 7,
   "name": "TOK2",
   "decimals": 4
  },
  "wartToAssetSwaps": [],
  "assetToWartSwaps": [
   {
    "inMempool": false,
    "txHash": "37bb2e782a32065a61962ae5e9637bfb8f45b5874db64a5b4bbd54c115d0a183",
    "limit": {
     "precExponent10": 4,
     "exponent2": -6,
     "mantissa": 64000,
     "hex": "fa0049",
     "doubleAdjusted": 0.1,
     "doubleRaw": 1000
    },
    "amount": {
     "str": "1000.0000",
     "u64": 10000000,
     "decimals": 4
    },
    "filled": {
     "str": "20.0010",
     "u64": 200010,
     "decimals": 4
    }
   },
   {
    "inMempool": false,
    "txHash": "c6fbade2df322654ece3bd7cb372a80812630a2478eff8db801bd89062d8f209",
    "limit": {
     "precExponent10": 4,
     "exponent2": -6,
     "mantissa": 64000,
     "hex": "fa0049",
     "doubleAdjusted": 0.1,
     "doubleRaw": 1000
    },
    "amount": {
     "str": "100.0000",
     "u64": 1000000,
     "decimals": 4
    },
    "filled": {
     "str": "0",
     "u64": 0,
     "decimals": 4
    }
   },
   {
    "inMempool": false,
    "txHash": "c116706c3a6682c5e3e358fbae00c0b1aab0a8e3cff1b5883ab037bcf567b9b8",
    "limit": {
     "precExponent10": 4,
     "exponent2": -5,
     "mantissa": 64000,
     "hex": "fa004a",
     "doubleAdjusted": 0.2,
     "doubleRaw": 2000
    },
    "amount": {
     "str": "300.0000",
     "u64": 3000000,
     "decimals": 4
    },
    "filled": {
     "str": "0",
     "u64": 0,
     "decimals": 4
    }
   },
   {
    "inMempool": false,
    "txHash": "c37cb4d58fdc54e281c172ab50cc9cb30fefddcfc8092d8793a52161a75b9dda",
    "limit": {
     "precExponent10": 4,
     "exponent2": -2,
     "mantissa": 40000,
     "hex": "9c404d",
     "doubleAdjusted": 1,
     "doubleRaw": 10000
    },
    "amount": {
     "str": "10000.0000",
     "u64": 100000000,
     "decimals": 4
    },
    "filled": {
     "str": "0",
     "u64": 0,
     "decimals": 4
    }
   }
  ],
  "liquidityPool": {
   "asset": {
    "str": "100.0000",
    "u64": 1000000,
    "decimals": 4
   },
   "wart": {
    "str": "1.00000000",
    "E8": 100000000
   },
   "shares": {
    "str": "0.10000000",
    "u64": 10000000,
    "decimals": 8
   }
  },
  "match": {
   "filled": {
    "base": {
     "str": "0",
     "u64": 0,
     "decimals": 4
    },
    "quote": {
     "str": "0",
     "E8": 0
    }
   },
   "toPool": {
    "isQuote": false,
    "amount": {
     "str": "0",
     "u64": 0,
     "decimals": 4
    }
   }
  }
 }
}
```
===

### `GET /account/:account/mempool`

Show mempool transactions for a specific account specified by either address or account ID. Same structure as [`/transaction/mempool`](#get-transactionmempool) but filtered to this account.

Example output:
```json
{
 "code": 0,
 "data": [
  {
   "type": "wartTransfer",
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

Show all open orders for a specific account across all markets. The account is specified by either address or account ID.

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

==- Example:
```json
{
  "code": 0,
  "data": {
    "baseAsset": { "decimals": 4, "hash": "0e4825ef...", "id": 7, "name": "TOK2" },
    "wartToAssetSwaps": [
      {
        "amount": { "decimals": 4, "str": "100.0000", "u64": 1000000 },
        "filled": { "decimals": 4, "str": "10.0000", "u64": 100000 },
        "inMempool": false,
        "limit": { "doubleAdjusted": 0.1, "doubleRaw": 1000.0, "exponent2": -6, "hex": "fa0049", "mantissa": 64000, "precExponent10": 4 },
        "txHash": "c6fbade2..."
      }
    ],
    "assetToWartSwaps": []
  }
}
```
===

### `GET /account/:account/balance/:tokenspec`

Show balance of an account for a specific asset. The `:account` parameter is the account address, and `:tokenspec` is the token spec (e.g., `asset:0e4825efffa294610d2ac376713e3bcc9b53d378e823834b64e5df01f75d3b0c`).

 #### Balance Types

 The API returns three balance types:

 | Type | Description |
 |------|-------------|
 | `total` | The total balance of the account |
 | `locked` | Balance locked in open orders |
 | `mempool` | Balance currently used by pending transactions in the mempool |

 Spendable amount is `total - locked` and can be reused for new transactions. When inserting new transactions, if `total - locked` is sufficient to cover the expenses required for the transaction but `total - locked - mempool` is not, the system may evict lower-fee transactions from the mempool to make room for a higher-fee transaction.

 Example output:

  ```json
  {
   "code": 0,
   "data": {
    "account": {
     "accountId": 9,
     "address": "2de77d5e23dc63e4c4149d394c979361e9e8e966"
    },
    "balance": {
     "locked": {
      "decimals": 4,
      "str": "12479.9990",
      "u64": 124799990
     },
     "mempool": {
      "decimals": 4,
      "str": "100.0000",
      "u64": 1000000
     },
     "total": {
      "decimals": 4,
      "str": "30899.9999",
      "u64": 308999999
     }
    },
    "lookupTrace": {
     "fails": [],
     "snapshotHeight": null
    },
    "token": {
     "decimals": 4,
     "id": 15,
     "name": "TOK2",
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

### `GET /account/:account/history/:beforeId`

Show transaction history of specific account (specified by address or account ID) with latest history ID smaller than specified in `:beforeId`. Use a very large number in this parameter to retrieve the latest history of the account. Returned data is structured as
```json
{
  "fromId": <integer>, // earliest returned history ID
  "perBlock": [...]    // array of actions per block
}
```
Every processed transaction is assigned an incrementally ascending history ID during block processing. The `fromId` field returns the earliest returned history ID. It serves as a bookmark position and can be used to iteratively retrieve earlier results in a loop. If no results are available, `fromId` is set to `0`.

The `perBlock` parameter has the same structure as the [`/transaction/latest`](#get-transactionlatest). Note that the last returned block is not necessarily complete, i.e. there might be more transactions in the same block that were not returned in this request.

==- Example output
```json
{
  "code": 0,
  "data": {
    "fromId": 1,
    "perBlock": [
      {
        "header": {
          "raw": "309c66716609a3257aeac8f317fa6dda728efb588d943c5b61726a2308a5a8fd077fffff5f99ee004dabcc4d2f4b604d99c69fecd261e8447c52043b8880acd8f535dc5a0000000469d65b8b00000000",
          "time": {
            "UTC": "2026-04-08 13:43:39 UTC",
            "timestamp": 1775655819
          },
          "target": "077fffff",
          "hash": "5797db9168c5b8bac134aa9bcea95f9bcd07ef3cb0f64dc31f49a96700177264",
          "pow": {
            "verusV2_2": true,
            "hashVerus": "6521b7cfb46cabb9f19891686ead024686b99dd5b44bab43f432c64133c94e98",
            "hashSha256t": "af1023b3084000bb686df8ba108ec5608d1d303addc1386b3fc4e3633f3fbdb2",
            "floatVerus": 0.39504574588499963,
            "floatSha256t": 0.6838400184642524
          },
          "merkleroot": "5f99ee004dabcc4d2f4b604d99c69fecd261e8447c52043b8880acd8f535dc5a",
          "nonce": "00000000",
          "prevHash": "309c66716609a3257aeac8f317fa6dda728efb588d943c5b61726a2308a5a8fd",
          "version": "00000004"
        },
        "body": {
          "reward": null,
          "wartTransfer": [
            {
              "transaction": {
                "data": {
                  "toAddress": "0000000000000000000000000000000000000000de47c9b2",
                  "amount": {
                    "str": "1.00000000",
                    "E8": 100000000
                  }
                },
                "hash": "897b3cc72950b41ee71527dc05a5bb6745801abbfb6179ad91dd0ccf2834eea8",
                "signedCommon": {
                  "originId": 1,
                  "originAddress": "3661579d61abde5837a8686dc4d65348a2fc61b1",
                  "fee": {
                    "str": "0.00009992",
                    "E8": 9992
                  },
                  "nonceId": 0,
                  "pinHeight": 0
                }
              },
              "historyId": 40
            }
          ],
          "tokenTransfer": [
            {
              "transaction": {
                "data": {
                  "toAddress": "0000000000000000000000000000000000000000de47c9b2",
                  "amount": {
                    "str": "99.9912",
                    "u64": 999912,
                    "decimals": 4
                  },
                  "asset": {
                    "hash": "f45b113119c7f7c000234f1090d5d181ab60b8b24526f1edd2f563aa1ca329f2",
                    "id": 2,
                    "name": "TOK2",
                    "decimals": 4
                  },
                  "isLiquidity": false,
                  "tokenSpec": "asset:f45b113119c7f7c000234f1090d5d181ab60b8b24526f1edd2f563aa1ca329f2"
                },
                "hash": "87a8f53490f32a29905554f4e23699774dd8c3e38c34033ba4812720824b5404",
                "signedCommon": {
                  "originId": 1,
                  "originAddress": "3661579d61abde5837a8686dc4d65348a2fc61b1",
                  "fee": {
                    "str": "0.00009992",
                    "E8": 9992
                  },
                  "nonceId": 1,
                  "pinHeight": 0
                }
              },
              "historyId": 41
            }
          ],
          "limitSwap": [],
          "match": [],
          "liquidityDeposit": [
            {
              "transaction": {
                "data": {
                  "baseAsset": {
                    "hash": "f45b113119c7f7c000234f1090d5d181ab60b8b24526f1edd2f563aa1ca329f2",
                    "id": 2,
                    "name": "TOK2",
                    "decimals": 4
                  },
                  "deposited": {
                    "base": {
                      "str": "99.9912",
                      "u64": 999912,
                      "decimals": 4
                    },
                    "quote": {
                      "str": "0.00999912",
                      "E8": 999912
                    }
                  }
                },
                "processed": {
                  "sharesReceived": {
                    "str": "99.9912",
                    "u64": 999912,
                    "decimals": 4
                  }
                },
                "hash": "47c50348397eed92b918501ec13c368e11df533388d199c895aebf57d9d8d0d3",
                "signedCommon": {
                  "originId": 1,
                  "originAddress": "3661579d61abde5837a8686dc4d65348a2fc61b1",
                  "fee": {
                    "str": "0.00009992",
                    "E8": 9992
                  },
                  "nonceId": 3,
                  "pinHeight": 0
                }
              },
              "historyId": 42
            }
          ],
          "liquidityWithdrawal": [],
          "assetCreation": [
            {
              "transaction": {
                "data": {
                  "name": "TOK2",
                  "supply": {
                    "str": "100000000.0000",
                    "u64": 1000000000000,
                    "decimals": 4
                  }
                },
                "processed": {
                  "assetId": 11
                },
                "hash": "d645efa5c0a64b409e44725d1d9b84ed7d800fca8ca5b3d71301d361652eddbc",
                "signedCommon": {
                  "originId": 1,
                  "originAddress": "3661579d61abde5837a8686dc4d65348a2fc61b1",
                  "fee": {
                    "str": "0.00009992",
                    "E8": 9992
                  },
                  "nonceId": 2,
                  "pinHeight": 0
                }
              },
              "historyId": 44
            }
          ],
          "cancelation": []
        },
        "confirmations": 5,
        "height": 21
      },
      {
        "header": {
          "raw": "cf132179a2e2895819572a0373eb880f03b0c026f5bbbd2fb30b913e628ada07077fffff561c2850138f1d76c87019ea1eeb0da7f9d50baa9a87c98f2260195d7c6d86ca0000000469524f7700000000",
          "time": {
            "UTC": "2025-12-29 09:52:55 UTC",
            "timestamp": 1767001975
          },
          "target": "077fffff",
          "hash": "5e6d289e5633edb1e5c93ff9156a225258c4b2057bfe86ce75c2830be5ea4b16",
          "pow": {
            "verusV2_2": true,
            "hashVerus": "c2c080f447fa0758c0a0d2415bcc8066b55f45e17bcb2f4f6c40ece46c13a69b",
            "hashSha256t": "fc8b272e7dbbe3fdd9235b32724f74af62ad1a3fa3cbaa68cd8fccea0b40a5c8",
            "floatVerus": 0.7607498737052083,
            "floatSha256t": 0.9864983069710433
          },
          "merkleroot": "561c2850138f1d76c87019ea1eeb0da7f9d50baa9a87c98f2260195d7c6d86ca",
          "nonce": "00000000",
          "prevHash": "cf132179a2e2895819572a0373eb880f03b0c026f5bbbd2fb30b913e628ada07",
          "version": "00000004"
        },
        "body": {
          "reward": null,
          "wartTransfer": [],
          "tokenTransfer": [],
          "limitSwap": [],
          "match": [],
          "liquidityDeposit": [
            {
              "transaction": {
                "data": {
                  "baseAsset": {
                    "hash": "f45b113119c7f7c000234f1090d5d181ab60b8b24526f1edd2f563aa1ca329f2",
                    "id": 2,
                    "name": "TOK2",
                    "decimals": 4
                  },
                  "deposited": {
                    "base": {
                      "str": "99.9912",
                      "u64": 999912,
                      "decimals": 4
                    },
                    "quote": {
                      "str": "0.00999912",
                      "E8": 999912
                    }
                  }
                },
                "processed": {
                  "sharesReceived": {
                    "str": "99.9912",
                    "u64": 999912,
                    "decimals": 4
                  }
                },
                "hash": "a3f6d1d3ee13cb8a85afb4dc28a3cbdb889c5c42ff1cc479469c4ff62fbca636",
                "signedCommon": {
                  "originId": 1,
                  "originAddress": "3661579d61abde5837a8686dc4d65348a2fc61b1",
                  "fee": {
                    "str": "0.00009992",
                    "E8": 9992
                  },
                  "nonceId": 25,
                  "pinHeight": 0
                }
              },
              "historyId": 12
            }
          ],
          "liquidityWithdrawal": [],
          "assetCreation": [],
          "cancelation": []
        },
        "confirmations": 18,
        "height": 8
      },
      {
        "header": {
          "raw": "9062306d5214ea3498efa06723741867777b4645818064b79402e898a4fc6cd5077fffff4532477af8b889ecae0c7c2673d9a0c9416bca6f5d5f299407ffc4c103994428000000046952494500000000",
          "time": {
            "UTC": "2025-12-29 09:26:29 UTC",
            "timestamp": 1767000389
          },
          "target": "077fffff",
          "hash": "cf132179a2e2895819572a0373eb880f03b0c026f5bbbd2fb30b913e628ada07",
          "pow": {
            "verusV2_2": true,
            "hashVerus": "bd055c1891f95d0a95ba24ba53e0e398dd0f638333c596b883be7141495e6ab8",
            "hashSha256t": "f1f8c6e37542f443540dffb4b7075128020a8e8f84c22d7c70b13db3a2668327",
            "floatVerus": 0.7383630331605673,
            "floatSha256t": 0.9452022842597216
          },
          "merkleroot": "4532477af8b889ecae0c7c2673d9a0c9416bca6f5d5f299407ffc4c103994428",
          "nonce": "00000000",
          "prevHash": "9062306d5214ea3498efa06723741867777b4645818064b79402e898a4fc6cd5",
          "version": "00000004"
        },
        "body": {
          "reward": null,
          "wartTransfer": [],
          "tokenTransfer": [],
          "limitSwap": [],
          "match": [],
          "liquidityDeposit": [],
          "liquidityWithdrawal": [],
          "assetCreation": [
            {
              "transaction": {
                "data": {
                  "name": "TOK2",
                  "supply": {
                    "str": "100000000.0000",
                    "u64": 1000000000000,
                    "decimals": 4
                  }
                },
                "processed": {
                  "assetId": 7
                },
                "hash": "0e4825efffa294610d2ac376713e3bcc9b53d378e823834b64e5df01f75d3b0c",
                "signedCommon": {
                  "originId": 1,
                  "originAddress": "3661579d61abde5837a8686dc4d65348a2fc61b1",
                  "fee": {
                    "str": "0.00009992",
                    "E8": 9992
                  },
                  "nonceId": 13,
                  "pinHeight": 0
                }
              },
              "historyId": 10
            }
          ],
          "cancelation": []
        },
        "confirmations": 19,
        "height": 7
      },
      {
        "header": {
          "raw": "0be65b99324fd4e798b2c633d0d98c990928775a4512e6226d4d08f84a9bc00d077fffff9b56be97d9af47584ca549d6c900299a67cdb4c4449b601572ed0e36f17da2af000000046952491e00000000",
          "time": {
            "UTC": "2025-12-29 09:25:50 UTC",
            "timestamp": 1767000350
          },
          "target": "077fffff",
          "hash": "f3e430a109ef514263ac7049920ffae76c4aab25b886d888f38de5d0ff0288c2",
          "pow": {
            "verusV2_2": true,
            "hashVerus": "dcaa7c25c2455c031f2d7bf06c073e74cf998e2cdb1a36957f4e385a5644a429",
            "hashSha256t": "5a5b94aa27d66b66b1286eadb03e13bc83526b9b4d888822b92e21166e5d958f",
            "floatVerus": 0.861976393731311,
            "floatSha256t": 0.3529599108733237
          },
          "merkleroot": "9b56be97d9af47584ca549d6c900299a67cdb4c4449b601572ed0e36f17da2af",
          "nonce": "00000000",
          "prevHash": "0be65b99324fd4e798b2c633d0d98c990928775a4512e6226d4d08f84a9bc00d",
          "version": "00000004"
        },
        "body": {
          "reward": null,
          "wartTransfer": [],
          "tokenTransfer": [],
          "limitSwap": [],
          "match": [],
          "liquidityDeposit": [],
          "liquidityWithdrawal": [],
          "assetCreation": [
            {
              "transaction": {
                "data": {
                  "name": "TOK2",
                  "supply": {
                    "str": "100000000.0000",
                    "u64": 1000000000000,
                    "decimals": 4
                  }
                },
                "processed": {
                  "assetId": 5
                },
                "hash": "58a74b758516d13e7f882922b483fca1627758c85b491e2cb9303e92e270ba88",
                "signedCommon": {
                  "originId": 1,
                  "originAddress": "3661579d61abde5837a8686dc4d65348a2fc61b1",
                  "fee": {
                    "str": "0.00009992",
                    "E8": 9992
                  },
                  "nonceId": 12,
                  "pinHeight": 0
                }
              },
              "historyId": 6
            }
          ],
          "cancelation": []
        },
        "confirmations": 22,
        "height": 4
      },
      {
        "header": {
          "raw": "e55ad07c558e15b5594c9cb00995c7f651e08c6b8c3fe9d83c2bc554dab7b074077fffff83f0e4680c3ae441ae6a357bf834034dff1eb8a9b7b424ba7b9672f386c38685000000046948247f00000000",
          "time": {
            "UTC": "2025-12-21 16:46:55 UTC",
            "timestamp": 1766335615
          },
          "target": "077fffff",
          "hash": "0be65b99324fd4e798b2c633d0d98c990928775a4512e6226d4d08f84a9bc00d",
          "pow": {
            "verusV2_2": true,
            "hashVerus": "dc106fb863fc979178b32145f8c9a5249587af6d146642b947b2011bea616ac1",
            "hashSha256t": "500c1d5d1c6f64b230c468aee35d197837268c2e8bab73ea863f3e446ff1729c",
            "floatVerus": 0.8596257995814085,
            "floatSha256t": 0.31268485565669835
          },
          "merkleroot": "83f0e4680c3ae441ae6a357bf834034dff1eb8a9b7b424ba7b9672f386c38685",
          "nonce": "00000000",
          "prevHash": "e55ad07c558e15b5594c9cb00995c7f651e08c6b8c3fe9d83c2bc554dab7b074",
          "version": "00000004"
        },
        "body": {
          "reward": {
            "transaction": {
              "data": {
                "toAddress": "3661579d61abde5837a8686dc4d65348a2fc61b1fe5f4093",
                "amount": {
                  "str": "3.00009992",
                  "E8": 300009992
                }
              },
              "hash": "49d16e7f40c9ca4d681bf00569e9d5ede9ad006d0123c7bdb665434e5c7bf03d"
            },
            "historyId": 3
          },
          "wartTransfer": [],
          "tokenTransfer": [],
          "limitSwap": [],
          "match": [],
          "liquidityDeposit": [],
          "liquidityWithdrawal": [],
          "assetCreation": [
            {
              "transaction": {
                "data": {
                  "name": "TOK2",
                  "supply": {
                    "str": "100000000.0000",
                    "u64": 1000000000000,
                    "decimals": 4
                  }
                },
                "processed": {
                  "assetId": 2
                },
                "hash": "f45b113119c7f7c000234f1090d5d181ab60b8b24526f1edd2f563aa1ca329f2",
                "signedCommon": {
                  "originId": 1,
                  "originAddress": "3661579d61abde5837a8686dc4d65348a2fc61b1",
                  "fee": {
                    "str": "0.00009992",
                    "E8": 9992
                  },
                  "nonceId": 11,
                  "pinHeight": 0
                }
              },
              "historyId": 4
            }
          ],
          "cancelation": []
        },
        "confirmations": 23,
        "height": 3
      },
      {
        "header": {
          "raw": "6851377f19b814ab3a4354c811a96908afba9c47cf3e4b85048d95de11e3f6aa077fffffa8e704f5d5485b743571f1742c2c4d7e83478d636a5a42f4cf255881ac7a062a000000036948247800000000",
          "time": {
            "UTC": "2025-12-21 16:46:48 UTC",
            "timestamp": 1766335608
          },
          "target": "077fffff",
          "hash": "e55ad07c558e15b5594c9cb00995c7f651e08c6b8c3fe9d83c2bc554dab7b074",
          "pow": {
            "verusV2_2": false,
            "hashVerus": "75de656c07275c4605af9c1b05b07fe9ee143f1d6e224a3097f4ae64a250f86b",
            "hashSha256t": "52776be4dd3b65ee93e0bbcb5e00e3ba2fb478e6d6eee777787faa268cd96d72",
            "floatVerus": 0.46042474638670683,
            "floatSha256t": 0.3221347266808152
          },
          "merkleroot": "a8e704f5d5485b743571f1742c2c4d7e83478d636a5a42f4cf255881ac7a062a",
          "nonce": "00000000",
          "prevHash": "6851377f19b814ab3a4354c811a96908afba9c47cf3e4b85048d95de11e3f6aa",
          "version": "00000003"
        },
        "body": {
          "reward": {
            "transaction": {
              "data": {
                "toAddress": "3661579d61abde5837a8686dc4d65348a2fc61b1fe5f4093",
                "amount": {
                  "str": "3.00000000",
                  "E8": 300000000
                }
              },
              "hash": "c870e807a46e6a5cf756165f95d511c7270a7dcd4eb51ddf7df047de3cfa1d8b"
            },
            "historyId": 2
          },
          "wartTransfer": [],
          "tokenTransfer": [],
          "limitSwap": [],
          "match": [],
          "liquidityDeposit": [],
          "liquidityWithdrawal": [],
          "assetCreation": [],
          "cancelation": []
        },
        "confirmations": 24,
        "height": 2
      },
      {
        "header": {
          "raw": "b29988793c29118a842ba38ea4ab13d43e3dce3f04511ba6ca3340542dfd8537077fffffeab3b88a101b745e8d7b530eb129a789f571a4718edc07435ae74998981a3a7f000000036948246d00000000",
          "time": {
            "UTC": "2025-12-21 16:46:37 UTC",
            "timestamp": 1766335597
          },
          "target": "077fffff",
          "hash": "6851377f19b814ab3a4354c811a96908afba9c47cf3e4b85048d95de11e3f6aa",
          "pow": {
            "verusV2_2": false,
            "hashVerus": "2443706cf33a161fd811fa69c7180787f4ddbc518f9e8cacd4efef29b3cc22a9",
            "hashSha256t": "353389ceb7bccd1efe58d3ff45b97c4efd99d0a460070d554a2f78edd92482b7",
            "floatVerus": 0.14165403973311186,
            "floatSha256t": 0.20781766204163432
          },
          "merkleroot": "eab3b88a101b745e8d7b530eb129a789f571a4718edc07435ae74998981a3a7f",
          "nonce": "00000000",
          "prevHash": "b29988793c29118a842ba38ea4ab13d43e3dce3f04511ba6ca3340542dfd8537",
          "version": "00000003"
        },
        "body": {
          "reward": {
            "transaction": {
              "data": {
                "toAddress": "3661579d61abde5837a8686dc4d65348a2fc61b1fe5f4093",
                "amount": {
                  "str": "3.00000000",
                  "E8": 300000000
                }
              },
              "hash": "da3dcfbc0dcb60be544a8510589f973b82651b48ddcb7943718dc3dcf93ae234"
            },
            "historyId": 1
          },
          "wartTransfer": [],
          "tokenTransfer": [],
          "limitSwap": [],
          "match": [],
          "liquidityDeposit": [],
          "liquidityWithdrawal": [],
          "assetCreation": [],
          "cancelation": []
        },
        "confirmations": 25,
        "height": 1
      }
    ]
  }
}
```
===

### `GET /account/richlist/:tokenspec`

Show richlist for a specific token. The `:tokenspec` has the form `asset:<asset hash>` or `liquidity:<asset hash>` to address either the balance of the asset itself or the associated liquidity in the corresponding pool with WART as quote currency.  Example output of `/account/richlist/asset:0e4825ef...`:

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
  "token": { "id": 7, "name": "TOK2", "decimals": 4, "spec": "asset:0e4825ef..." }
 }
}
```

### `GET /peers/ip_count`

Show number of connections per IP address.

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

Show the connection schedule for peers. Returns one of two variants: `TCPConnectionSchedule` for native nodes (with `connectedVerified`, `disconnectedVerified`, and `feelers` fields) or `WSConnectionSchedule` for browser nodes (empty object for now).

==- Example for native nodes:
```json
{
  "code": 0,
  "data": {
    "connectedVerified": [
      {
        "schedule": {
          "address": "tcp://51.222.248.202:9186",
          "expiresIn": null,
          "lastError": null,
          "sleepDuration": 3600
        },
        "secondsSinceVerified": 42
      }
    ],
    "disconnectedVerified": [
      {
        "schedule": {
          "address": "tcp://213.199.59.252:20016",
          "expiresIn": 86400,
          "lastError": null,
          "sleepDuration": 7200
        },
        "secondsSinceVerified": 3600
      }
    ],
    "feelers": [
      {
        "address": "tcp://65.21.76.159:9186",
        "expiresIn": null,
        "lastError": "connection refused",
        "sleepDuration": 300
      }
    ]
  }
}
```
===

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

### `GET /chart/candles/:asset/:interval`

Show OHLCV candle data for a specific asset and interval accepting query parameters `from`, `to` and `n`.

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

Returns an array of candles, where each candle is an 8-element array: `[begin_timestamp, height, open, high, low, close, base, quote]`.

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

### `GET /chart/trades/:asset`

Show historic trade data for a specific asset accepting query parameters `from`, `to` and `n`.

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

Returns an array of trades, where each trade is a 4-element array: `[height, timestamp, base, quote]`.

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

### `GET /chart/hashrate/block/:from/:to/:window`

Show hashrate chart data over a block range. `:from` and `:to` are block heights, `:window` is the averaging window in blocks.

==- Example:
```json
{
  "code": 0,
  "data": {
    "range": { "begin": 1949600, "end": 1949760 },
    "data": [140000000000000, 141000000000000, 139500000000000]
  }
}
```
===

### `GET /chart/hashrate/time/:from/:to/:interval`

Show hashrate chart data over a time range. `:from` and `:to` are Unix timestamps, `:interval` is the bucket size in seconds.

==- Example:
```json
{
  "code": 0,
  "data": {
    "range": { "begin": 1727200000, "end": 1727286400 },
    "interval": 3600,
    "data": [
      { "hashrate": 140000000000000, "height": 1949700, "timestamp": 1727200000 },
      { "hashrate": 141000000000000, "height": 1949720, "timestamp": 1727203600 }
    ]
  }
}
```
===

### `GET /tools/encode16bit/from_e8/:feeE8`

 Round raw fee integer representation (coin amount is this number divided by 10^8) to closest 16 bit representation. This is required for fee specification in the `/transaction/add` endpoint. Fee rounding is also implemented in the [official client libraries](../libraries.md).

!!!warning Note
The recommended way to create 16 bit encoded fee values for transaction generation is to implement the rounding method on the client side, see [Fee Encoding](./rest/create-transaction.md#fee-encoding). This tool is a fallback method for quick prototyping.
!!!

Example output of `/tools/encode16bit/from_e8/5002`

```json
{
  "code": 0,
  "data": {
    "original": { "E8": 5002, "str": "0.00005002" },
    "rounded": { "E8": 5000, "str": "0.00005000", "bytes": "1388" }
  }
}
```

### `GET /tools/encode16bit/from_string/:string`

 Round fee amount string to closest 16 bit representation. This is required for fee specification in the `/transaction/add` endpoint.

!!!warning Note
The recommended way to create 16 bit encoded fee values for transaction generation is to implement the rounding method on the client side, see [Fee Encoding](./rest/create-transaction.md#fee-encoding). This tool is a fallback method for quick prototyping.
!!!

Example output of `/tools/encode16bit/from_string/0.001`:

```json
{
  "code": 0,
  "data": {
    "original": { "E8": 100000, "str": "0.001" },
    "rounded": { "E8": 99968, "str": "0.00099968", "bytes": "4220" }
  }
}
```

### `GET /tools/parse_price/:price/:decimals`

Parse price adjusted for the specified number of decimals. The `doubleAdjusted` property is the human-readable representation of the parsed price.

Example output of `/tools/parse_price/0.123456/3`:

```json
{
 "code": 0,
 "data": {
  "decimals": 3,
  "floor": {
   "precExponent10": 5,
   "exponent2": -2,
   "mantissa": 49382,
   "hex": "c0e64d",
   "doubleAdjusted": 0.12345500000000001,
   "doubleRaw": 12345.5
  },
  "ceil": {
   "precExponent10": 5,
   "exponent2": -2,
   "mantissa": 49383,
   "hex": "c0e74d",
   "doubleAdjusted": 0.12345750000000001,
   "doubleRaw": 12345.75
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

!!!warning Warning
This endpoint should only be used for testing purposes. For security reasons the node should not be involved in wallet-related operations involving a private key.
!!!

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

Generate fake mining data using the burn address `0000000000000000000000000000000000000000de47c9b2`. This operation is against the protocol and other nodes will ban your node.

```json
{ "code": 0 }
```

### `GET /debug/rollback`

Rollback the chain by one block.

```json
{ "code": 0 }
```

### `GET /debug/fakemine/:address`

Generate fake mining data for a specific address. This operation is aginst the protocol and other nodes will ban your node.

```json
{ "code": 0 }
```
