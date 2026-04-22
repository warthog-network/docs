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
The content of the `data` field depends on the transaction type specified in the `type` field. For some transaction types there exists the field `processed` which is null if and only if `mined` is null, i.e. if the transaction is still in mempool. See [Transactions](./rest/transactions.md) for details.
==- Example output of `/transaction/lookup/50f860f2c626c002aaac9db9b1db566ba280b10fc24fa470728b0b4ed88008f9`:

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
    "hegiht": 11,
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
Each block in the returned `perBlock` array has the structure
```json
{
 "actions":{
  "reward": ... // null or reward
  "wartTransfers": [...],
  "tokenTransfers": [...],
  "newOrders": [...],
  "matches": [...],
  "liquidityDeposits": [...],
  "liquidityWithdrawals": [...],
  "assetCreations": [...],
  "cancelations": [...]
 }
}
```
All entries above in `actions` are of the structure `{ "historyId": ..., "transaction": ...}` where `historyId` is an unsigned 64 bit integer and `transaction` has the structure
```json
{
 "data": ...,           // transaction specific data
 "hash": "f364da99...", // transaction hash
 "processed": ...,      // this field is present only for some transaction types, holds processed data
 "signedCommon": ...    // signature related data
}
```
See [Transactions](./rest/transactions.md) for details. In contrast to the `/transaction/lookup/:txid` endpoint, the `processed` field, if it is present, can never be `null` because we return mined transactions in this endpoint whereas the transaction returned by `/transaction/lookup/:txid` may still be in mempool which is exactly the case where `null` is returned for `processed`.

==- Example output of `/transaction/latest`:
```json
{
 "code": 0,
 "data": {
  "perBlock": [
   {
    "height": 1,
    "confirmations": 25,
    "actions": {
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
     "wartTransfers": [],
     "tokenTransfers": [],
     "newOrders": [],
     "matches": [],
     "liquidityDeposits": [],
     "liquidityWithdrawals": [],
     "assetCreations": [],
     "cancelations": []
    }
   },
   {
    "height": 2,
    "confirmations": 24,
    "actions": {
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
     "wartTransfers": [],
     "tokenTransfers": [],
     "newOrders": [],
     "matches": [],
     "liquidityDeposits": [],
     "liquidityWithdrawals": [],
     "assetCreations": [],
     "cancelations": []
    }
   },
   {
    "height": 3,
    "confirmations": 23,
    "actions": {
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
     "wartTransfers": [],
     "tokenTransfers": [],
     "newOrders": [],
     "matches": [],
     "liquidityDeposits": [],
     "liquidityWithdrawals": [],
     "assetCreations": [
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
     "cancelations": []
    }
   },
   {
    "height": 4,
    "confirmations": 22,
    "actions": {
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
     "wartTransfers": [],
     "tokenTransfers": [],
     "newOrders": [],
     "matches": [],
     "liquidityDeposits": [],
     "liquidityWithdrawals": [],
     "assetCreations": [
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
     "cancelations": []
    }
   },
   {
    "height": 5,
    "confirmations": 21,
    "actions": {
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
     "wartTransfers": [],
     "tokenTransfers": [],
     "newOrders": [],
     "matches": [],
     "liquidityDeposits": [],
     "liquidityWithdrawals": [],
     "assetCreations": [],
     "cancelations": []
    }
   },
   {
    "height": 6,
    "confirmations": 20,
    "actions": {
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
     "wartTransfers": [],
     "tokenTransfers": [],
     "newOrders": [],
     "matches": [],
     "liquidityDeposits": [],
     "liquidityWithdrawals": [],
     "assetCreations": [],
     "cancelations": []
    }
   },
   {
    "height": 7,
    "confirmations": 19,
    "actions": {
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
     "wartTransfers": [],
     "tokenTransfers": [],
     "newOrders": [],
     "matches": [],
     "liquidityDeposits": [],
     "liquidityWithdrawals": [],
     "assetCreations": [
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
     "cancelations": []
    }
   },
   {
    "height": 8,
    "confirmations": 18,
    "actions": {
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
     "wartTransfers": [],
     "tokenTransfers": [],
     "newOrders": [],
     "matches": [],
     "liquidityDeposits": [
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
     "liquidityWithdrawals": [],
     "assetCreations": [],
     "cancelations": []
    }
   },
   {
    "height": 9,
    "confirmations": 17,
    "actions": {
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
     "wartTransfers": [],
     "tokenTransfers": [],
     "newOrders": [],
     "matches": [],
     "liquidityDeposits": [],
     "liquidityWithdrawals": [],
     "assetCreations": [],
     "cancelations": []
    }
   },
   {
    "height": 10,
    "confirmations": 16,
    "actions": {
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
     "wartTransfers": [],
     "tokenTransfers": [],
     "newOrders": [
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
     "matches": [
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
     "liquidityDeposits": [],
     "liquidityWithdrawals": [],
     "assetCreations": [],
     "cancelations": []
    }
   },
   {
    "height": 11,
    "confirmations": 15,
    "actions": {
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
     "wartTransfers": [],
     "tokenTransfers": [],
     "newOrders": [
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
     "matches": [
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
     "liquidityDeposits": [],
     "liquidityWithdrawals": [],
     "assetCreations": [],
     "cancelations": []
    }
   },
   {
    "height": 12,
    "confirmations": 14,
    "actions": {
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
     "wartTransfers": [],
     "tokenTransfers": [],
     "newOrders": [
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
     "matches": [
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
     "liquidityDeposits": [],
     "liquidityWithdrawals": [],
     "assetCreations": [],
     "cancelations": []
    }
   },
   {
    "height": 13,
    "confirmations": 13,
    "actions": {
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
     "wartTransfers": [],
     "tokenTransfers": [],
     "newOrders": [
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
     "matches": [
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
     "liquidityDeposits": [],
     "liquidityWithdrawals": [],
     "assetCreations": [],
     "cancelations": []
    }
   },
   {
    "height": 14,
    "confirmations": 12,
    "actions": {
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
     "wartTransfers": [],
     "tokenTransfers": [],
     "newOrders": [
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
     "matches": [
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
     "liquidityDeposits": [],
     "liquidityWithdrawals": [],
     "assetCreations": [],
     "cancelations": []
    }
   },
   {
    "height": 15,
    "confirmations": 11,
    "actions": {
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
     "wartTransfers": [],
     "tokenTransfers": [],
     "newOrders": [],
     "matches": [],
     "liquidityDeposits": [],
     "liquidityWithdrawals": [],
     "assetCreations": [],
     "cancelations": []
    }
   },
   {
    "height": 16,
    "confirmations": 10,
    "actions": {
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
     "wartTransfers": [],
     "tokenTransfers": [],
     "newOrders": [],
     "matches": [],
     "liquidityDeposits": [],
     "liquidityWithdrawals": [],
     "assetCreations": [],
     "cancelations": []
    }
   },
   {
    "height": 17,
    "confirmations": 9,
    "actions": {
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
     "wartTransfers": [],
     "tokenTransfers": [],
     "newOrders": [],
     "matches": [],
     "liquidityDeposits": [],
     "liquidityWithdrawals": [],
     "assetCreations": [],
     "cancelations": []
    }
   },
   {
    "height": 18,
    "confirmations": 8,
    "actions": {
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
     "wartTransfers": [],
     "tokenTransfers": [],
     "newOrders": [],
     "matches": [],
     "liquidityDeposits": [],
     "liquidityWithdrawals": [],
     "assetCreations": [],
     "cancelations": []
    }
   },
   {
    "height": 19,
    "confirmations": 7,
    "actions": {
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
     "wartTransfers": [],
     "tokenTransfers": [],
     "newOrders": [],
     "matches": [],
     "liquidityDeposits": [],
     "liquidityWithdrawals": [],
     "assetCreations": [],
     "cancelations": []
    }
   },
   {
    "height": 20,
    "confirmations": 6,
    "actions": {
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
     "wartTransfers": [],
     "tokenTransfers": [],
     "newOrders": [
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
     "matches": [
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
     "liquidityDeposits": [],
     "liquidityWithdrawals": [],
     "assetCreations": [],
     "cancelations": []
    }
   },
   {
    "height": 21,
    "confirmations": 5,
    "actions": {
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
     "wartTransfers": [
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
     "tokenTransfers": [
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
     "newOrders": [],
     "matches": [],
     "liquidityDeposits": [
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
     "liquidityWithdrawals": [],
     "assetCreations": [
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
     "cancelations": []
    }
   },
   {
    "height": 22,
    "confirmations": 4,
    "actions": {
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
     "wartTransfers": [],
     "tokenTransfers": [],
     "newOrders": [],
     "matches": [],
     "liquidityDeposits": [],
     "liquidityWithdrawals": [],
     "assetCreations": [],
     "cancelations": []
    }
   },
   {
    "height": 23,
    "confirmations": 3,
    "actions": {
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
     "wartTransfers": [],
     "tokenTransfers": [],
     "newOrders": [],
     "matches": [],
     "liquidityDeposits": [],
     "liquidityWithdrawals": [],
     "assetCreations": [],
     "cancelations": []
    }
   },
   {
    "height": 24,
    "confirmations": 2,
    "actions": {
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
     "wartTransfers": [],
     "tokenTransfers": [],
     "newOrders": [],
     "matches": [],
     "liquidityDeposits": [],
     "liquidityWithdrawals": [],
     "assetCreations": [],
     "cancelations": []
    }
   },
   {
    "height": 25,
    "confirmations": 1,
    "actions": {
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
     "wartTransfers": [],
     "tokenTransfers": [],
     "newOrders": [],
     "matches": [],
     "liquidityDeposits": [],
     "liquidityWithdrawals": [],
     "assetCreations": [],
     "cancelations": []
    }
   }
  ],
  "fromId": 1
 }
}
```
===

For detailed JSON structure of each action type, see [!ref Block Actions](rest/block-actions.md).
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

### `GET /dex/market/:asset`

Show orders, liquidity pool and matching for the market with specified base asset and WART as quote asset. The `:asset` parameter can be the asset hash or the asset id. The returned structure is as follows:

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

Show mempool transactions for a specific account. Same structure as `GET /transaction/mempool` but filtered to this account.

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
