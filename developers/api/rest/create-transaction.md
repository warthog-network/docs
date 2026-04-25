# Creating a New Transaction

!!!
This documentation was partially generated with AI augmentation. Report inconsistencies at [github.com/warthog-network/docs/issues](https://github.com/warthog-network/docs/issues).
!!!

To create a new transaction, use the `POST /transaction/add` API method. POST data must be in JSON format and describes the transaction data in a JSON object. There must always exist a `type` string property that describes the transaction type which determines the other required parameters.

## Transaction Types

| Transaction Type | `type` Value | Description |
|------------------|--------------|-------------|
| WART Transfer | `wartTransfer` | Transfer WART tokens |
| Token Transfer | `tokenTransfer` | Transfer a specific asset or liquidity token |
| Limit Swap | `limitSwap` | Create a buy or sell limit order |
| Liquidity Deposit | `liquidityDeposit` | Deposit liquidity into an asset's WART pool |
| Liquidity Withdrawal | `liquidityWithdrawal` | Withdraw liquidity by redeeming tokens |
| Cancelation | `cancelation` | Cancel a pending transaction or order |
| Asset Creation | `assetCreation` | Create a new asset |

## Common Parameters

The following parameters are common to all transaction types:

| Parameter | Type | Description |
|-----------|------|-------------|
| `type` | string | Transaction type (see table above) |
| `feeE8` | uint64 (optional) | Transaction fee in WART multiplied by 10^8. Must be exactly representable in 16-bit encoding. |
| `feeStr` | string (optional) | Transaction fee as string (e.g., `"0.0001"`). Must be exactly representable in 16-bit encoding. |
| `nonceId` | uint32 | Unique identifier for this transaction at the given `pinHeight`. |
| `pinHeight` | uint32 | Block height whose hash is included in the signature. |
| `signature65` | string (130 chars) | Hex-encoded 65-byte compact recoverable ECDSA signature. |

!!!
Either `feeE8` or `feeStr` must be specified. The fee must be exactly representable in a 16-bit encoding (see [Fee Encoding](#fee-encoding)).
!!!

## Transaction Types

==- WART Transfer

Transfer WART tokens to another address.

**Parameters:**

| Parameter | Type | Description |
|-----------|------|-------------|
| `type` | string | Set to `wartTransfer` |
| `toAddr` | string (48 chars) | Recipient address |
| `wartE8` | uint64 (optional) | Amount of WART to send multiplied by 10^8 |
| `wartStr` | string (optional) | Amount of WART to send (e.g., `"1.5"` or `"1.50000000"`) |

!!!
Either `wartE8` or `wartStr` must be specified.
!!!

**Example Request:**

```json
{
  "type": "wartTransfer",
  "toAddr": "848b08b803e95640f8cb30af1b3166701b152b98c2cd70ee",
  "wartStr": "1.5",
  "feeStr": "0.0001",
  "nonceId": 1,
  "pinHeight": 1198931,
  "signature65": "..."
}
```

**Signature Hash Bytes:**

| Bytes | Description |
|-------|-------------|
| 1-32 | `pinHash` (hash of block at height `pinHeight`) |
| 33-36 | `pinHeight` (uint32, network byte order) |
| 37-40 | `nonceId` (uint32, network byte order) |
| 41-43 | `reserved` (3 zero bytes) |
| 44-51 | `feeE8` (uint64, network byte order) |
| 52-71 | `toAddr` (20 bytes, without checksum) |
| 72-79 | `wartE8` (uint64, network byte order) |

---

==- Token Transfer

Transfer a specific asset or its pool's liquidity token.

**Parameters:**

| Parameter | Type | Description |
|-----------|------|-------------|
| `type` | string | Set to `tokenTransfer` |
| `assetHash` | string (64 chars) | Hash of the asset to transfer |
| `isLiquidity` | boolean | `false` for asset, `true` for liquidity tokens |
| `toAddr` | string (48 chars) | Recipient address |
| `amountU64` | uint64 | Number of smallest representable units |

!!!warning
The `amountU64` specifies the smallest units. The conversion to token units depends on the asset's precision `p` (one token unit = `10^p` smallest units). Assets have their own precision, while liquidity tokens always have precision 8 (like WART).
!!!

**Example Request:**

```json
{
  "type": "tokenTransfer",
  "assetHash": "0e4825efffa294610d2ac376713e3bcc9b53d378e823834b64e5df01f75d3b0c",
  "isLiquidity": false,
  "toAddr": "848b08b803e95640f8cb30af1b3166701b152b98c2cd70ee",
  "amountU64": 1000000,
  "feeStr": "0.0001",
  "nonceId": 1,
  "pinHeight": 1198931,
  "signature65": "..."
}
```

**Signature Hash Bytes:**

| Bytes | Description |
|-------|-------------|
| 1-32 | `pinHash` |
| 33-36 | `pinHeight` (uint32, network byte order) |
| 37-40 | `nonceId` (uint32, network byte order) |
| 41-43 | `reserved` (3 zero bytes) |
| 44-51 | `feeE8` (uint64, network byte order) |
| 52-83 | `assetHash` (32 bytes) |
| 84 | `isLiquidity` (1 byte: 1 for `true`, 0 for `false`) |
| 85-104 | `toAddr` (20 bytes, without checksum) |
| 105-112 | `amountU64` (uint64, network byte order) |

---

==- Limit Swap

Create a new buy or sell limit order for a specific asset.

**Parameters:**

| Parameter | Type | Description |
|-----------|------|-------------|
| `type` | string | Set to `limitSwap` |
| `assetHash` | string (64 chars) | Hash of the asset to trade |
| `isBuy` | boolean | `true` for buy order, `false` for sell order |
| `amountU64` | uint64 | Amount to spend (WART if buying, asset if selling) |
| `limit` | string (6 chars) | Hex-encoded 3-byte limit price (use `/tools/parse_price`) |

!!!warning
For buy orders, `amountU64` is in WART (precision 8). For sell orders, it's in the asset's smallest units (precision varies by asset).
!!!

**Example Request:**

```json
{
  "type": "limitSwap",
  "assetHash": "0e4825efffa294610d2ac376713e3bcc9b53d378e823834b64e5df01f75d3b0c",
  "isBuy": true,
  "amountU64": 100000000,
  "limit": "c0e74d",
  "feeStr": "0.0001",
  "nonceId": 1,
  "pinHeight": 1198931,
  "signature65": "..."
}
```

**Signature Hash Bytes:**

| Bytes | Description |
|-------|-------------|
| 1-32 | `pinHash` |
| 33-36 | `pinHeight` (uint32, network byte order) |
| 37-40 | `nonceId` (uint32, network byte order) |
| 41-43 | `reserved` (3 zero bytes) |
| 44-51 | `feeE8` (uint64, network byte order) |
| 52-83 | `assetHash` (32 bytes) |
| 84 | `isBuy` (1 byte: 1 for `true`, 0 for `false`) |
| 85-92 | `amountU64` (uint64, network byte order) |
| 93-95 | `limit` (3 bytes) |

---

==- Liquidity Deposit

Deposit liquidity into an asset's WART pool.

**Parameters:**

| Parameter | Type | Description |
|-----------|------|-------------|
| `type` | string | Set to `liquidityDeposit` |
| `assetHash` | string (64 chars) | Hash of the asset |
| `amountU64` | uint64 | Amount of asset to deposit (smallest units) |
| `wartE8` | uint64 (optional) | Amount of WART to deposit multiplied by 10^8 |
| `wartStr` | string (optional) | Amount of WART to deposit |

!!!
Either `wartE8` or `wartStr` must be specified.
!!!

**Example Request:**

```json
{
  "type": "liquidityDeposit",
  "assetHash": "0e4825efffa294610d2ac376713e3bcc9b53d378e823834b64e5df01f75d3b0c",
  "amountU64": 1000000,
  "wartStr": "10.0",
  "feeStr": "0.0001",
  "nonceId": 1,
  "pinHeight": 1198931,
  "signature65": "..."
}
```

**Signature Hash Bytes:**

| Bytes | Description |
|-------|-------------|
| 1-32 | `pinHash` |
| 33-36 | `pinHeight` (uint32, network byte order) |
| 37-40 | `nonceId` (uint32, network byte order) |
| 41-43 | `reserved` (3 zero bytes) |
| 44-51 | `feeE8` (uint64, network byte order) |
| 52-83 | `assetHash` (32 bytes) |
| 84-91 | `amountU64` (uint64, network byte order) |
| 92-99 | `wartE8` (uint64, network byte order) |

---

==- Liquidity Withdrawal

Withdraw liquidity from an asset's WART pool by redeeming liquidity tokens.

**Parameters:**

| Parameter | Type | Description |
|-----------|------|-------------|
| `type` | string | Set to `liquidityWithdrawal` |
| `assetHash` | string (64 chars) | Hash of the asset |
| `amountE8` | uint64 | Amount of liquidity tokens to redeem (multiplied by 10^8) |

**Example Request:**

```json
{
  "type": "liquidityWithdrawal",
  "assetHash": "0e4825efffa294610d2ac376713e3bcc9b53d378e823834b64e5df01f75d3b0c",
  "amountE8": 1000000000,
  "feeStr": "0.0001",
  "nonceId": 1,
  "pinHeight": 1198931,
  "signature65": "..."
}
```

**Signature Hash Bytes:**

| Bytes | Description |
|-------|-------------|
| 1-32 | `pinHash` |
| 33-36 | `pinHeight` (uint32, network byte order) |
| 37-40 | `nonceId` (uint32, network byte order) |
| 41-43 | `reserved` (3 zero bytes) |
| 44-51 | `feeE8` (uint64, network byte order) |
| 52-83 | `assetHash` (32 bytes) |
| 84-91 | `amountE8` (uint64, network byte order) |

---

==- Cancelation

Cancel a pending transaction or an existing order.

**Parameters:**

| Parameter | Type | Description |
|-----------|------|-------------|
| `type` | string | Set to `cancelation` |
| `cancelHeight` | uint32 | The `pinHeight` of the transaction to cancel |
| `cancelNonceId` | uint32 | The `nonceId` of the transaction to cancel |

**Example Request:**

```json
{
  "type": "cancelation",
  "cancelHeight": 1198900,
  "cancelNonceId": 5,
  "feeStr": "0.0001",
  "nonceId": 1,
  "pinHeight": 1198931,
  "signature65": "..."
}
```

**Signature Hash Bytes:**

| Bytes | Description |
|-------|-------------|
| 1-32 | `pinHash` |
| 33-36 | `pinHeight` (uint32, network byte order) |
| 37-40 | `nonceId` (uint32, network byte order) |
| 41-43 | `reserved` (3 zero bytes) |
| 44-51 | `feeE8` (uint64, network byte order) |
| 52-55 | `cancelHeight` (uint32, network byte order) |
| 56-59 | `cancelNonceId` (uint32, network byte order) |

---

==- Asset Creation

Create a new asset.

**Parameters:**

| Parameter | Type | Description |
|-----------|------|-------------|
| `type` | string | Set to `assetCreation` |
| `name` | string (1-5 chars) | Name of the asset |
| `precision` | uint8 (0-18) | Precision `p` (one unit = `10^p` smallest units) |
| `supplyU64` | uint64 | Total supply in smallest units |

**Example Request:**

```json
{
  "type": "assetCreation",
  "name": "TOK1",
  "precision": 8,
  "supplyU64": 1000000000000,
  "feeStr": "0.0001",
  "nonceId": 1,
  "pinHeight": 1198931,
  "signature65": "..."
}
```

**Signature Hash Bytes:**

| Bytes | Description |
|-------|-------------|
| 1-32 | `pinHash` |
| 33-36 | `pinHeight` (uint32, network byte order) |
| 37-40 | `nonceId` (uint32, network byte order) |
| 41-43 | `reserved` (3 zero bytes) |
| 44-51 | `feeE8` (uint64, network byte order) |
| 52-59 | `supplyU64` (uint64, network byte order) |
| 60 | `precision` (1 byte) |
| 61-65 | `name` (5 bytes, null-padded) |

---
===

## Fee Encoding

Transaction fees are not subtracted from the transfer amount. The sender spends both the transfer amount and the fee, while `toAddr` receives only the transferred amount.

For efficiency, fees are encoded as 2-byte floating-point numbers (16 bits):
- First 6 bits: exponent
- Remaining 10 bits: 11-bit mantissa (with implicit leading 1)

Only 64-bit values exactly representable in 16 bits are accepted. 

The recommended way is to round on client side, see the [official client libraries](../libraries.md) for reference. For quick prototyping use these endpoints to round arbitrary values on node side:

- `GET /tools/encode16bit/from_e8/:feeE8` - Round from numeric value
- `GET /tools/encode16bit/from_string/:string` - Round from string

## Signature Generation

The sender's address is recovered from the ECDSA signature `signature65`. The signature is created with the private key corresponding to the sender's address.

### Steps

1. **Get pinHash:** Call `GET /chain/head` and extract `pinHash` and `pinHeight`.

2. **Create transaction hash:** Compute SHA256 of the transaction-specific byte sequence (see each transaction type above).

3. **Generate signature:** Create a secp256k1-ECDSA recoverable signature with:
   - `r`: 32-byte coordinate parameter
   - `s`: 32-byte coordinate parameter (normalized to lower value)
   - `recid`: 1-byte recovery ID (0, 1, 2, or 3)

4. **Encode signature:** Concatenate to form the 65-byte signature:

   | Bytes | Value |
   |-------|-------|
   | 1-32 | `r` |
   | 33-64 | `s` |
   | 65 | `recid` |

!!!
Warthog uses a custom signature format where the recovery ID is the last byte without the standard offset of 27.
!!!

## Response

On success, returns the transaction hash:

```json
{
  "code": 0,
  "data": {
    "txHash": "f364da997bf7a3c3bc8ead22041509228d6bdea39fcbcdc6be56030433d54219"
  }
}
```

## Integration Guides

Working code examples for generating and sending transactions in [Python3, NodeJS, and Elixir](../../integration/wallets.md).

## Libraries
[!ref](../../libraries.md)
