# POST /transaction/add
 Send transactions as JSON object. This endpoint is for all transaction types.
#### Supported Transactions
The structure depends on the `type` parameter.
==- Wart Transfer
 Transfer `WART`.

 PARAMETER | TYPE | DETAILS
 ----------|------|--------
`type`| string | Set to `wartTransfer`.
 `feeE8` (optional) | unsigned 64 bit integer | Amount of `WART` to spend on transaction fees multiplied by 10^8. For example to send one 0.00000001 `WART` this value must be 1. This value must be exactly representable value in a 16 bit encoding, see below.
 `fee` (optional) | string | Amount of `WART` to spend on transaction fees. This value must be exactly representable value in a 16 bit encoding, see below.
 `nonceId`   | unsigned 32 bit integer | To avoid double spend, there can only be one transaction with a specific (pinHeight,nonceId) pair. The same nonceId can be used for different pinHeight values.
 `pinHeight` | unsigned 32 bit integer | Signature includes block hash at this height
 `signature65`| string of length 130 | hex-encoded 65 byte compact recoverable ECDSA signature in custom format, see below.
 `toAddr`    | string of length 48| The address that `WART` shall be transferred to.
 `wartE8` (optional) | unsigned 64 bit integer | Amount of `WART` to send multiplied by 10^8. For example to send one coin this value must be 100000000.
 `wartStr` (optional) | string | Amount of `WART` to send. Format must be string, non-scientific notation with at most 8 digits after comma. "`1`" or "`1.00000000`" are valid.

**NOTE**:

- The `WART` amount must be specified either via `wart` or via `wartE8`.
- The transaction fee must be specified either via `fee` or via `feeE8`.
- Miner fee must be exactly representable in a 16 bit encoding.

The transaction hash needed for signature generation is the SHA256 hash of the following bytes:

BYTES | DESCRIPTION
------|------------
1 -32 | `pinHash` (hash of block at height `pinHeight`)
33-36 | `pinHeight` (`uint32_t` in network byte order)
37-40 | `nonceId` (`uint32_t` in network byte order)
41-43 | `reserved` (3 bytes containing 0)
44-51 | `feeE8` (`uint64_t` in network byte order)
52-71 | `toAddr` receiving address (20 bytes without the final 4 byte checksum)
72-79 | `wartE8` (`uint64_t` in network byte order)
==- Token Transfer
 Transfer a specific asset or its pool's liquidity token.

 PARAMETER | TYPE | DETAILS
 ----------|------|--------
`type`| string | Set to `tokenTransfer`.
 `amountU64` | unsigned 64 bit integer | Number of smallest representable units of the token to be sent. See the warning below.
 `assetHash` | string of length 64 | The asset hash specifies the asset for this transfer.
 `feeE8` (optional) | unsigned 64 bit integer | Amount of `WART` to spend on transaction fees multiplied by 10^8. For example to send one 0.00000001 `WART` this value must be 1. This value must be exactly representable value in a 16 bit encoding, see below.
 `fee` (optional) | string | Amount of `WART` to spend on transaction fees. This value must be exactly representable value in a 16 bit encoding, see below.
 `isLiquidity` | boolean | Specify whether you want to send the asset itself (`false`) or its pool liquidity (`true`).
 `nonceId`   | unsigned 32 bit integer | To avoid double spend, there can only be one transaction with a specific (pinHeight,nonceId) pair. The same nonceId can be used for different pinHeight values.
 `pinHeight` | unsigned 32 bit integer | Signature includes block hash at this height
 `signature65`| string of length 130 | hex-encoded 65 byte compact recoverable ECDSA signature in custom format, see below.
 `toAddr`    | string of length 48| The address that tokens shall be transferred to.

!!!warning Warning
The `amountU64` parameter specifies the number of smallest units of the specified token type. The actual conversion to token units depends on the token type's precision `p` (one token unit corresponds to `10^p` smallest representable units). Note that every asset has **its own precision** fixed during asset creation whereas any asset pool's liquidity **always has precision 8** (like the WART token type).
!!!
**NOTE**:

- The transaction fee must be specified either via `fee` or via `feeE8`.
- The transaction fee must be specified either via `fee` or via `feeE8`.
- Miner fee must be exactly representable in a 16 bit encoding.

The transaction hash needed for signature generation is the SHA256 hash of the following bytes:

BYTES | DESCRIPTION
------|------------
1 -32 | `pinHash` (hash of block at height `pinHeight`)
33-36 | `pinHeight` (`uint32_t` in network byte order)
37-40 | `nonceId` (`uint32_t` in network byte order)
41-43 | `reserved` (3 bytes containing 0)
44-51 | `feeE8` (`uint64_t` in network byte order)
52-83 | `assetHash` asset hash (hash of asset)
84-84 | `isLiquidity` (1 byte bool, 1 for `true`, 0 for `false`)
85-104 | `toAddr` receiving address (20 bytes without the final 4 byte checksum)
105-112 | `amountU64` (`uint64_t` in network byte order)

==- Limit Swap
Create a new buy or sell limit swap order for a specific asset.

 PARAMETER | TYPE | DETAILS
 ----------|------|--------
`type`| string | Set to `limitSwap`.
 `amountU64` | unsigned 64 bit integer | Number of smallest representable units of the token to be spent on the swap.
 `assetHash` | string of length 64 | The asset hash specifies the asset for this transfer.
 `feeE8` (optional) | unsigned 64 bit integer | Amount of coins to spend on transaction fees multiplied by 10^8. For example to send one 0.00000001 coins this value must be 1. This value must be exactly representable value in a 16 bit encoding, see below.
 `fee` (optional) | string | Amount of coins to spend on transaction fees. This value must be exactly representable value in a 16 bit encoding, see below.
 `isBuy` | boolean | Specify whether this order is a buy (`true`) or a sell (`false`) order.
 `limit`    | string of length 6| Hexadecimal of of 3 byte encoded limit price. Use the `/tools/parse_price` endpoint to request this encoding.
 `nonceId`   | unsigned 32 bit integer | To avoid double spend, there can only be one transaction with a specific (pinHeight,nonceId) pair. The same nonceId can be used for different pinHeight values
 `pinHeight` | unsigned 32 bit integer | Signature includes block hash at this height
 `signature65`| string of length 130 | hex-encoded 65 byte compact recoverable ECDSA signature in custom format, see below.

!!!warning Warning
The `amountU64` parameter specifies the number of smallest units of the token that is converted. Depending on the `isBuy` parameter, this is either `WART` or the specified asset. The actual conversion to token units depends on the token type's precision `p` (one token unit corresponds to `10^p` smallest representable units). Note that **`WART` has precision 8** whereas the asset has **its own precision** fixed during asset creation.
!!!
**NOTE**:

- The transaction fee must be specified either via `fee` or via `feeE8`.
- Miner fee must be exactly representable in a 16 bit encoding.

The transaction hash needed for signature generation is the SHA256 hash of the following bytes:

BYTES | DESCRIPTION
------|------------
1 -32 | `pinHash` (hash of block at height `pinHeight`)
33-36 | `pinHeight` (`uint32_t` in network byte order)
37-40 | `nonceId` (`uint32_t` in network byte order)
41-43 | `reserved` (3 bytes containing 0)
44-51 | `feeE8` (`uint64_t` in network byte order)
52-83 | `assetHash` asset hash (hash of asset)
84-84 | `buy` (1 byte bool, 1 for `true`, 0 for `false`)
85-92 | `amountU64` (`uint64_t` in network byte order)
93-95 | `limit` (3 byte encoding from `/tools/parse_price`)
96-116 | `toAddr` receiving address (20 bytes without the final 4 byte checksum)

==- Liquidity Deposit
Deposit liquidity into an asset's `WART` pool.

 PARAMETER | TYPE | DETAILS
 ----------|------|--------
`type`| string | Set to `liquidityDeposit`.
 `amountU64` | unsigned 64 bit integer | Number of smallest representable units of the asset to spend into the pool. See the warning below.
 `assetHash` | string of length 64 | The asset hash specifies the asset for this transfer.
 `feeE8` (optional) | unsigned 64 bit integer | Amount of coins to spend on transaction fees multiplied by 10^8. For example to send one 0.00000001 coins this value must be 1. This value must be exactly representable value in a 16 bit encoding, see below.
 `fee` (optional) | string | Amount of coins to spend on transaction fees. This value must be exactly representable value in a 16 bit encoding, see below.
 `nonceId`   | unsigned 32 bit integer | To avoid double spend, there can only be one transaction with a specific (pinHeight,nonceId) pair. The same nonceId can be used for different pinHeight values
 `pinHeight` | unsigned 32 bit integer | Signature includes block hash at this height
 `signature65`| string of length 130 | hex-encoded 65 byte compact recoverable ECDSA signature in custom format, see below.
 `wartE8` (optional) | unsigned 64 bit integer | Amount of coins to send multiplied by 10^8. For example to send one coin this value must be 100000000.
 `wartStr` (optional) | string | Amount of coins to send. Format must be string, non-scientific notation with at most 8 digits after comma. "`1`" or "`1.00000000`" are valid.

!!!warning Warning
The `amountU64` parameter specifies the number of smallest units of the specified token type. The actual conversion to token units depends on the assets precision `p` (one token unit corresponds to `10^p` smallest representable units). Note that every asset has **its own precision** fixed during asset creation.
!!!
**NOTE**:

- The `WART` amount must be specified either via `wart` or via `wartE8`.
- The transaction fee must be specified either via `fee` or via `feeE8`.
- Miner fee must be exactly representable in a 16 bit encoding.

The transaction hash needed for signature generation is the SHA256 hash of the following bytes:

BYTES | DESCRIPTION
------|------------
1 -32 | `pinHash` (hash of block at height `pinHeight`)
33-36 | `pinHeight` (`uint32_t` in network byte order)
37-40 | `nonceId` (`uint32_t` in network byte order)
41-43 | `reserved` (3 bytes containing 0)
44-51 | `feeE8` (`uint64_t` in network byte order)
52-83 | `assetHash` asset hash (hash of asset)
84-91 | `amountU64` (`uint64_t` in network byte order)
92-99 | `wartE8` (`uint64_t` in network byte order)

==- Liquidity Withdrawal
Withdraw liquidity from an asset's `WART` pool by redeeming liquidity tokens.

 PARAMETER | TYPE | DETAILS
 ----------|------|--------
`type`| string | Set to `liquidityDeposit`.
 `amountE8` | unsigned 64 bit integer | Number of smallest representable units of the liquidity tokens to spend into the pool. Since liquidity tokens always have precision 8, this is the number of liquidity tokens multiplied by `10^8`.
 `assetHash` | string of length 64 | The asset hash specifies the asset for this transfer.
 `feeE8` (optional) | unsigned 64 bit integer | Amount of coins to spend on transaction fees multiplied by 10^8. For example to send one 0.00000001 coins this value must be 1. This value must be exactly representable value in a 16 bit encoding, see below.
 `fee` (optional) | string | Amount of coins to spend on transaction fees. This value must be exactly representable value in a 16 bit encoding, see below.
 `nonceId`   | unsigned 32 bit integer | To avoid double spend, there can only be one transaction with a specific (pinHeight,nonceId) pair. The same nonceId can be used for different pinHeight values
 `pinHeight` | unsigned 32 bit integer | Signature includes block hash at this height
 `signature65`| string of length 130 | hex-encoded 65 byte compact recoverable ECDSA signature in custom format, see below.

!!!warning Warning
The `amountU64` parameter specifies the number of smallest units of the specified token type. The actual conversion to token units depends on the assets precision `p` (one token unit corresponds to `10^p` smallest representable units). Note that every asset has **its own precision** fixed during asset creation.
!!!
**NOTE**:

- The transaction fee must be specified either via `fee` or via `feeE8`.
- Miner fee must be exactly representable in a 16 bit encoding.

The transaction hash needed for signature generation is the SHA256 hash of the following bytes:

BYTES | DESCRIPTION
------|------------
1 -32 | `pinHash` (hash of block at height `pinHeight`)
33-36 | `pinHeight` (`uint32_t` in network byte order)
37-40 | `nonceId` (`uint32_t` in network byte order)
41-43 | `reserved` (3 bytes containing 0)
44-51 | `feeE8` (`uint64_t` in network byte order)
52-83 | `assetHash` asset hash (hash of asset)
84-91 | `amountE8` (`uint64_t` in network byte order)

==- Cancelation
Cancel a pending transaction or an existing order.

 PARAMETER | TYPE | DETAILS
 ----------|------|--------
`type`| string | Set to `liquidityDeposit`.
 `cancelHeight` | unsigned 32 bit integer | The canceled transaction's `pinHeight`.
 `cancelNonceId` | unsinged 32 bit integer | The canceled transaction's `nonceId`.
 `feeE8` (optional) | unsigned 64 bit integer | Amount of coins to spend on transaction fees multiplied by 10^8. For example to send one 0.00000001 coins this value must be 1. This value must be exactly representable value in a 16 bit encoding, see below.
 `fee` (optional) | string | Amount of coins to spend on transaction fees. This value must be exactly representable value in a 16 bit encoding, see below.
 `nonceId`   | unsigned 32 bit integer | To avoid double spend, there can only be one transaction with a specific (pinHeight,nonceId) pair. The same nonceId can be used for different pinHeight values
 `pinHeight` | unsigned 32 bit integer | Signature includes block hash at this height
 `signature65`| string of length 130 | hex-encoded 65 byte compact recoverable ECDSA signature in custom format, see below.

**NOTE**:

- The transaction fee must be specified either via `fee` or via `feeE8`.
- Miner fee must be exactly representable in a 16 bit encoding.

The transaction hash needed for signature generation is the SHA256 hash of the following bytes:

BYTES | DESCRIPTION
------|------------
1 -32 | `pinHash` (hash of block at height `pinHeight`)
33-36 | `pinHeight` (`uint32_t` in network byte order)
37-40 | `nonceId` (`uint32_t` in network byte order)
41-43 | `reserved` (3 bytes containing 0)
44-51 | `feeE8` (`uint64_t` in network byte order)
52-55 | `cancelHeight` (`uint32_t` in network byte order)
56-59 | `cancelNonceId` (`uint32_t` in network byte order)

==- AssetCreations
Create a new asset.

 PARAMETER | TYPE | DETAILS
 ----------|------|--------
`type`| string | Set to `liquidityDeposit`.
 `feeE8` (optional) | unsigned 64 bit integer | Amount of coins to spend on transaction fees multiplied by 10^8. For example to send one 0.00000001 coins this value must be 1. This value must be exactly representable value in a 16 bit encoding, see below.
 `fee` (optional) | string | Amount of coins to spend on transaction fees. This value must be exactly representable value in a 16 bit encoding, see below.
 `name` | string of length 1 to 5 | The name of asset to be created.
 `nonceId`   | unsigned 32 bit integer | To avoid double spend, there can only be one transaction with a specific (pinHeight,nonceId) pair. The same nonceId can be used for different pinHeight values
 `pinHeight` | unsigned 32 bit integer | Signature includes block hash at this height
 `precision` | values from 0 to 18 | Precision `p` of the asset to be created. One asset unit will correspond to `10^p` smallest representable units.
 `signature65`| string of length 130 | hex-encoded 65 byte compact recoverable ECDSA signature in custom format, see below.
 `supplyU64` | unsinged 64 bit integer | Number of smallest representable units of the asset to be created. The supply in asset units is `supplyU64 / 10^precision`.

**NOTE**:

- The transaction fee must be specified either via `fee` or via `feeE8`.
- Miner fee must be exactly representable in a 16 bit encoding.

The transaction hash needed for signature generation is the SHA256 hash of the following bytes:

BYTES | DESCRIPTION
------|------------
1 -32 | `pinHash` (hash of block at height `pinHeight`)
33-36 | `pinHeight` (`uint32_t` in network byte order)
37-40 | `nonceId` (`uint32_t` in network byte order)
41-43 | `reserved` (3 bytes containing 0)
44-51 | `feeE8` (`uint64_t` in network byte order)
52-59 | `supplyU64` (`uint64_t` in network byte order)
60-60 | `precision`
61-65 | `name` (back-padded to 5 bytes with 0)
===


#### Details on fees

Fees are not subtracted from the amount sent in the transaction. The sender spends both, transfer amount and transaction fee, `toAddr` receives the transferred amount and the miner of the block including this transaction gets the transaction fee.

For efficiency and compactness transaction fees are internally encoded as 2-byte floating-point numbers (16 bits), where the first 6 bits encode the exponent and the remaining 10 bits encode a 11 bit mantissa starting with an implicit 1. Of course not every 64 bit value can be encoded in 16 bits and only fee 64 bit values which are representable exactly in the 16 bits encoding are accepted. You can use the `/tools/encode16bit/from_e8/:feeE8` or `/tools/encode16bit/from_string/:feestring` endpoints to round an arbitrary 64-bit fee value to an accepted 64 bit value.

#### How to specify the sender?

The sender's address is recovered from the recoverable ECDSA signature `signature65`. It is implicitly specified by creating a signature with the corresponding private key.

#### Signature generation

The following steps are required:

1. Call the `/chain/head` endpoint and extract the  `pinHash` and `pinHeight` fields.

2. Generate the secp256k1-ECDSA recoverable signature of the 32-byte transaction hash using the private key corresponding to the sender's address. The signature will have three properties:

- `r`: 32 byte coordinate parameter
- `s`: 32 byte coordinate parameter
- `recid`: 1 byte recovery id, it should automatically have one of the four values 0,1,2,3.

3. Concatenate the parameters to form the 65-byte compact normalized (lower `s`) recoverable signature with the following byte structure:

BYTES | DESCRIPTION
------|------------
1 -32 | `r`
33-64 | `s`
65    | `recid`

Note that this is not the standard compact recoverable signature representation because in Warthog, the recoverable id is the last byte of the 65 byte signature and has no offset of 27.

#### Integration guides

We provide working code snippets on how to generate and send transactions [in Python3](./integration_python.md), [Elixir](./integration_elixir.md) and [NodeJS](./integration_nodejs.md).
