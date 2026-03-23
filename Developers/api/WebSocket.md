---
title: WebSocket
---
# WebSocket API

The WebSocket endpoint allows real-time data streams on several topics such as `chain`, `account` and `connection` related information. Subscribe with a message containing `"action": "subscribe"` and unsubscribe similarly with `"action": "unsubscribe"`, as shown below.

## Endpoint

```
WEBSOCKET /stream
```

## Chain

Subscribe to 10 latest blocks. Upon subscription, the 10 latest blocks are provided in a `chain.state` message. When chain is appended (no rollback), *either* incremental updates are given (i.e. new blocks) as a `chain.append` message *or* (in case of more than 10 new blocks) the latest 10 blocks are provided in a `chain.state` message. When chain is appended with rollback, a `chain.fork` message is sent over the websocket connection.

**Subscribe**
```json
{"action": "subscribe", "topic": "chain"}
```

**Event types**

### 1. `chain.state`

Receive 10 latest blocks.

```json
{
  "latestBlocks": [
    {
      "body": {
        "rewards": [
          {
            "amount": "3.00000000",
            "amountE8": 300000000,
            "toAddress": "f07353f22a59c6a98042f29ec87d1fc0f44a6c3ceea24797",
            "txHash": "fe33577de3186c1869f9084bfe0aeaee2e92c8fe10f87eedb438036b341b98ef"
          }
        ],
        "transfers": []
      },
      "confirmations": 11,
      "header": {
        "difficulty": 14142941042437.885,
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
        "timestamp": 1727201019,
        "utc": "2024-09-24 18:03:39 UTC",
        "version": "00000003"
      },
      "height": 1949766,
      "timestamp": 1727201019,
      "utc": "2024-09-24 18:03:39 UTC"
    },
    ...
  ],
  "eventName": "chain.state",
  "head": {
    "difficulty": 14142941042437.885,
    "hash": "cbaa8aefc5c5df6a4cfa8134bb6780895c0389f40a8c0f8ca9c627e1177ea2df",
    "height": 1949776,
    "is_janushash": true,
    "pinHash": "568e30a9abdbc510c9a21ec2dcb636e66cd4d1fb60028a31c65906069114e988",
    "pinHeight": 1949760,
    "worksum": 29360669698622464000,
    "worksumHex": "0x00000000000000000000000000000000000000000000000197760d8809f796a0"
  }
}
```

### 2. `chain.append`

```json
{
  "eventName": "chain.append",
  "head": {
    "difficulty": 14142941042437.885,
    "hash": "1edd2043a92ac4747487aecaa7dc8145b6bf8ae4509c3bae4e4d9ad9727fe4ee",
    "height": 1950121,
    "is_janushash": true,
    "pinHash": "ae8e35d5e0ab86b4a30612b37717e29ea2d4d21f039abe130fcb94b03079e073",
    "pinHeight": 1950112,
    "worksum": 29365549013281480000,
    "worksumHex": "0x0000000000000000000000000000000000000000000000019787633e02f726a0"
  },
  "newBlocks": {
    "data": [
      {
        "body": {
          "rewards": [],
          "transfers": []
        },
        "confirmations": 0,
        "header": {
          "difficulty": 14142941042437.885,
          "hash": "1edd2043a92ac4747487aecaa7dc8145b6bf8ae4509c3bae4e4d9ad9727fe4ee",
          "merkleroot": "dcfeba982307da35c0098709c20f4ea0a4f04e0bbcd1b0c34c8ecc30157159eb",
          "nonce": "1aace551",
          "pow": {
            "floatSha256t": 0.012002777308225632,
            "floatVerus": 6.341241019439662e-13,
            "hashSha256t": "03129d3047448ed01936f206d4a5f59de702a35df450d36706ef93545cf03c36",
            "hashVerus": "0000000000b27d750524493e994e98b7dea83af149ae7673dfc3d21935c1ae05",
            "verusV2.2": true
          },
          "prevHash": "8c8811ee10a9c5c7d23f6f4791cbf03b781531574d53f224278cb78531a4c2fe",
          "raw": "8c8811ee10a9c5c7d23f6f4791cbf03b781531574d53f224278cb78531a4c2fe0ae7cde7dcfeba982307da35c0098709c20f4ea0a4f04e0bbcd1b0c34c8ecc30157159eb0000000366f31e3c1aace551",
          "target": "0ae7cde7",
          "timestamp": 1727209020,
          "utc": "2024-09-24 20:17:00 UTC",
          "version": "00000003"
        },
        "height": 1950121,
        "timestamp": 1727209020,
        "utc": "2024-09-24 20:17:00 UTC"
      }
    ]
  }
}
```

### 3. `chain.fork`

```json
{
  "eventName": "chain.fork",
  "head": {
    "difficulty": 14142941042437.885,
    "hash": "8bb4c08b71a496921d121460c55041678943c89107a59306fda948dfc2c8dc8a",
    "height": 1950129,
    "is_janushash": true,
    "pinHash": "ae8e35d5e0ab86b4a30612b37717e29ea2d4d21f039abe130fcb94b03079e073",
    "pinHeight": 1950112,
    "worksum": 29365662156809806000,
    "worksumHex": "0x0000000000000000000000000000000000000000000000019787ca254ac7a6a0"
  },
  "latestBlocks": {
    "data": [
      {
        "body": {
          "rewards": [
            {
              "amount": "3.00000000",
              "amountE8": 300000000,
              "toAddress": "e4f3052d8c5858ddeca667b2130eec5886c8e9d7fcdea3ec",
              "txHash": "a19c2b9410440e569c670ca223504379aa09697f7b061e8dbf04c2a7870fd9b1"
            }
          ],
          "transfers": []
        },
        "confirmations": 11,
        "header": {
          "difficulty": 14142941042437.885,
          "hash": "10b5ed6868f0e5c4fbd859a8dd5a68d7cf4c9038025acde37a004e9f02e7c85e",
          "merkleroot": "9205c8127aabd555acb7ff3f95733bd6ec19cb110c2878c55ae77b893fdec7ec",
          "nonce": "46f6a275",
          "pow": {
            "floatSha256t": 0.012585171731188893,
            "floatVerus": 3.504885418321763e-15,
            "hashSha256t": "0338c8258ed30e6088b4b81cba45f5c40553be12af0323abb7418e2cf63becd7",
            "hashVerus": "000000000000fc8db96d150937429ce192e0ad69458311c4d8415b720e59b6c0",
            "verusV2.2": true
          },
          "prevHash": "30537a679be7d6082cc905be4a717ea7985f0853403a40dedea1be4e224916dd",
          "raw": "30537a679be7d6082cc905be4a717ea7985f0853403a40dedea1be4e224916dd0ae7cde79205c8127aabd555acb7ff3f95733bd6ec19cb110c2878c55ae77b893fdec7ec0000000366f31e0c46f6a275",
          "target": "0ae7cde7",
          "timestamp": 1727208972,
          "utc": "2024-09-24 20:16:12 UTC",
          "version": "00000003"
        },
        "height": 1950119,
        "timestamp": 1727208972,
        "utc": "2024-09-24 20:16:12 UTC"
      },
      ...
    ]
  },
  "rollbackLength": 1950127
}
```

## Account

Subscribe to account updates such as new transactions and balance changes. Address is specified in the `message["params"]["address"]` field. To unsubscribe, this parameter has be specified again.

**Subscribe**
```json
{"action": "subscribe", "topic": "account", "params": {"address": "f07353f22a59c6a98042f29ec87d1fc0f44a6c3ceea24797"}}
```

**Event types**

### 1. `account.state`

```json
{
  "address": "f07353f22a59c6a98042f29ec87d1fc0f44a6c3ceea24797",
  "balance": "51557.99973680",
  "balanceE8": 5155799973680,
  "eventName": "account.state",
  "history": []
}
```

### 2. `account.delta`

```json
{
  "address": "f07353f22a59c6a98042f29ec87d1fc0f44a6c3ceea24797",
  "balance": "51551.99973680",
  "balanceE8": 5155199973680,
  "eventName": "account.delta",
  "history": {
    "data": [
      {
        "body": {
          "rewards": [
            {
              "amount": "3.00000000",
              "amountE8": 300000000,
              "toAddress": "f07353f22a59c6a98042f29ec87d1fc0f44a6c3ceea24797",
              "txHash": "fe33577de3186c1869f9084bfe0aeaee2e92c8fe10f87eedb438036b341b98ef"
            }
          ],
          "transfers": []
        },
        "confirmations": 0,
        "header": {
          "difficulty": 14142941042437.885,
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
          "timestamp": 1727201019,
          "utc": "2024-09-24 18:03:39 UTC",
          "version": "00000003"
        },
        "height": 1949766,
        "timestamp": 1727201019,
        "utc": "2024-09-24 18:03:39 UTC"
      }
    ]
  }
}
```

## Connection

Subscribe to connection state and deltas. After subscription, a `connection.state` message is sent. New connections trigger a `connection.add` message while removed connections trigger a `connection.remove` message.

**Subscribe**
```json
{"action": "subscribe", "topic": "connection"}
```

**Event types**

### 1. `connection.state`

```json
{
  "connections": [
    {
      "id": 22,
      "inbound": false,
      "peerAddr": "tcp://213.199.59.252:20016",
      "since": 0
    },
    ...
    {
      "id": 1261,
      "inbound": false,
      "peerAddr": "tcp://51.222.248.202:9186",
      "since": 0
    }
  ],
  "eventName": "connection.state",
  "total": 16
}
```

### 2. `connection.add`

```json
{
  "connection": {
    "id": 83,
    "inbound": false,
    "peerAddr": "tcp://65.21.76.159:9186",
    "since": 0
  },
  "eventName": "connection.add",
  "total": 3
}
```

### 3. `connection.remove`

```json
{
  "eventName": "connection.remove",
  "id": 83,
  "total": 2
}
```
