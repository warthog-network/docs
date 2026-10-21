---
label: Wallets
---
# Wallet Integration

In this guide we demonstrate how to handle wallets and send transactions in Python 3, NodeJS and Elixir.
Note that we also provide [SDKs](../SDKs.md).

## Prerequisites

+++ Python
We use the `coincurve`, `pycryptodome` and `requests` Python packages.
+++ Node JS
We use the `secp256k1`, `node:crypto` and `sync-request` NodeJS packages.
+++ Elixir
We use the `{:curvy, "~> 0.3.1"}`, `{:jason, "~> 1.4"}` and `{:httpoison, "~> 2.0"}` Hex packages.
+++

## Handling wallets

Below we demonstrate how to:
- generate a new private key,
- load a private key from hexadecimal encoding,
- derive a public key from a private key,
- derive a raw Warthog address from a public key, and
- extend the raw Warthog address by its checksum to obtain an ordinary Warthog address.

+++ Python
```python
from coincurve import PrivateKey
import secrets

# secp256k1 group order, a private key must be in [1, n - 1]
ORDER = 0xFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFEBAAEDCE6AF48A03BBFD25E8CD0364141

##############################
# Handling wallets
##############################

# generate private key
private_key = secrets.token_bytes(32)
while not 1 <= int.from_bytes(private_key, 'big') < ORDER:
    private_key = secrets.token_bytes(32)
pk = PrivateKey(private_key)

# alternatively read private key
pkhex = '966a71a98bb5d13e9116c0dffa3f1a7877e45c6f563897b96cfd5c59bf0803e0'
pk = PrivateKey(bytes.fromhex(pkhex))
private_key = pk.secret

# print private key
print("private_key:")
print(private_key.hex())

# derive public key
pubkey = pk.public_key.format(compressed=True)

# print public key
print("public key:")
print(pubkey.hex())

# convert public key to raw address
from Crypto.Hash import RIPEMD160, SHA256
sha = SHA256.SHA256Hash(pubkey).digest()
addr_raw = RIPEMD160.RIPEMD160Hash(sha).digest()

# generate address by appending checksum
addr_hash = SHA256.SHA256Hash(addr_raw).digest()
checksum = addr_hash[0:4]
addr = addr_raw + checksum

# print address
print("address:", addr.hex())
```
+++ Node JS
```javascript
// var, not const: the transaction examples below require secp256k1 again,
// and redeclaring a const in the same scope is a SyntaxError.
var secp256k1 = require('secp256k1');
const crypto = require('node:crypto');

//////////////////////////////
// Handling wallets
//////////////////////////////

// generate private key
var privateKey = crypto.randomBytes(32);
while (!secp256k1.privateKeyVerify(privateKey)) {
  privateKey = crypto.randomBytes(32);
}
var pkhex = privateKey.toString("hex")

// alternatively read private key
var pkhex = '966a71a98bb5d13e9116c0dffa3f1a7877e45c6f563897b96cfd5c59bf0803e0'

// print private key
console.log("private key:", pkhex)

// derive public key
var pubKey = Buffer.from(secp256k1.publicKeyCreate(Buffer.from(pkhex, "hex"), true)).toString("hex")

// print public key
console.log("public key:", pubKey)

// convert public key to raw address
var sha = crypto.createHash('sha256').update(Buffer.from(pubKey,"hex")).digest()
var addrRaw = crypto.createHash('ripemd160').update(sha).digest()

// generate address by appending checksum
var checksum = crypto.createHash('sha256').update(addrRaw).digest().slice(0,4)
var addr = Buffer.concat([addrRaw, checksum]).toString("hex")

// print address
console.log("address:", addr)

// helper functions for buffer construction
function uint32BE(value) {
    const buf = Buffer.alloc(4);
    buf.writeUInt32BE(value, 0);
    return buf;
}

function uint64BE(value) {
    const buf = Buffer.alloc(8);
    buf.writeBigUInt64BE(BigInt(value), 0);
    return buf;
}
```
+++ Elixir
```elixir
##############################
# Handling wallets
##############################

# generate private key
pk = Curvy.generate_key()

# alternatively read private key
pk = "966a71a98bb5d13e9116c0dffa3f1a7877e45c6f563897b96cfd5c59bf0803e0" |> Base.decode16!(case: :lower) |> Curvy.Key.from_privkey()

# print private key
Curvy.Key.to_privkey(pk) |> Base.encode16 |> IO.puts

# derive public key
pubkey = Curvy.Key.to_pubkey(pk)

# print public key
pubkey |> Base.encode16 |> IO.puts

# convert public key to raw address
addr_raw = :crypto.hash(:ripemd160, :crypto.hash(:sha256, pubkey))

# generate address by appending checksum
<<checksum::binary-size(4), _::binary>> = :crypto.hash(:sha256, addr_raw)
addr = addr_raw <> checksum

# print address
IO.puts("address: #{addr |> Base.encode16()}")
```
+++

## Transaction Concepts

In Warthog, transactions are pinned to a specific block on the blockchain. The height of this block is its `pinHeight`, and its hash is the `pinHash`. This information must be retrieved from the node before generating a transaction.

This serves two purposes:
1. Users can be sure that transactions will not be pending forever.
2. By choosing a particular `pinHeight`, users can actively control the time to live (TTL) of a pending transaction before it is discarded.

A transaction's signature is based on a *chain pin*, which is a block hash and height at a multiple of 32. When included in a block, the height information is encoded as a single byte which is interpreted as negative offset times 32 with respect to the block's height floored to multiples of 32. When verifying a transaction's signature, the transaction's chain pin has to be restored, i.e. the node computes the chain pin from the block's height and the encoded offset and retrieves the block hash at that position from its own chain.

Since the encoded offset can only represent 256 different positions on the grid of heights at multiples of 32, no heights before 8160 blocks from the current block's floored height (roughly 45 hours at the 20 seconds block time) can be addressed technically, and a transaction pinned below that point can no longer be included in any block. This represents an elegant way to enforce a time-to-live (TTL) for transactions. For convenience, the `/chain/head` endpoint provides the latest `pinHeight` value together with the corresponding block hash `pinHash`.

## WART Transfer Transaction

Transfer WART tokens to another address.

+++ Python
```python
import requests
import json
baseurl = "http://localhost:3000"

# get pinHash and pinHeight from warthog node
head_raw = requests.get(baseurl + '/chain/head').content
head = json.loads(head_raw)
# the V1 API (mainnet) returns the pin fields at the top level. the V2 API
# (DeFi node) nests them: head["data"]["chainHead"]["pinHash"] and ["pinHeight"]
pinHash = head["data"]["pinHash"]
pinHeight = head["data"]["pinHeight"]

# send parameters
nonceId = 0  # 32 bit number, unique per pinHash and pinHeight
toAddr = '0000000000000000000000000000000000000000de47c9b2'  # burn destination address
wartE8 = 100000000  # 1 WART, this must be an integer, coin amount * 10^8

# round fee from WART amount
rawFee = "0.00009999"  # WARNING: NO SCIENTIFIC NOTATION
result = requests.get(baseurl + '/tools/encode16bit/from_string/' + rawFee).content
encode16bit_result = json.loads(result)
feeE8 = encode16bit_result["data"]["rounded"]["E8"]

# generate bytes to sign
to_sign = (
    bytes.fromhex(pinHash) +
    pinHeight.to_bytes(4, byteorder='big') +
    nonceId.to_bytes(4, byteorder='big') +
    b'\x00\x00\x00' +
    feeE8.to_bytes(8, byteorder='big') +
    bytes.fromhex(toAddr)[0:20] +
    wartE8.to_bytes(8, byteorder='big')
)

# create signature
from coincurve import PrivateKey
from hashlib import sha256
digest = sha256(to_sign).digest()

# sign with recovery id, RFC 6979 deterministic. sign_recoverable returns the
# 65 bytes r || s || recid, and libsecp256k1 already normalizes to low-s.
signature65 = PrivateKey(private_key).sign_recoverable(digest, hasher=None)

# post transaction request to warthog node
postdata = {
    "type": "wartTransfer",
    "pinHeight": pinHeight,
    "nonceId": nonceId,
    "toAddr": toAddr,
    "wartE8": wartE8,
    "feeE8": feeE8,
    "signature65": signature65.hex()
}
rep = requests.post(baseurl + "/transaction/add", json=postdata)
print(rep.content)
```
+++ Node JS
```javascript
//////////////////////////////
// WART Transfer Transaction
//////////////////////////////
const { createHash } = require('crypto');
var request = require("sync-request");
var secp256k1 = require("secp256k1");

var baseurl = "http://localhost:3000";

// get pinHash and pinHeight from warthog node
var head = JSON.parse(request("GET", baseurl + "/chain/head").body.toString());
// the V1 API (mainnet) returns the pin fields at the top level. the V2 API
// (DeFi node) nests them: head.data.chainHead.pinHeight and head.data.chainHead.pinHash
var pinHeight = head.data.pinHeight;
var pinHash = head.data.pinHash;

// send parameters
var nonceId = 0;  // 32 bit number, unique per pinHash and pinHeight
var toAddr = "0000000000000000000000000000000000000000de47c9b2";  // burn destination address
var wartE8 = 100000000;  // 1 WART, this must be an integer, coin amount * 10^8

// round fee from WART amount
var rawFee = "0.00009999";  // WARNING: NO SCIENTIFIC NOTATION
var result = request("GET", baseurl + "/tools/encode16bit/from_string/" + rawFee).body.toString();
var encode16bit_result = JSON.parse(result);
var feeE8 = encode16bit_result["data"]["rounded"]["E8"];

// generate bytes to sign
var toSign = Buffer.concat([
    Buffer.from(pinHash, "hex"),
    uint32BE(pinHeight),
    uint32BE(nonceId),
    Buffer.alloc(3),
    uint64BE(feeE8),
    Buffer.from(toAddr.slice(0, 40), "hex"),
    uint64BE(wartE8)
]);

// sign with recovery id
var signHash = createHash("sha256").update(toSign).digest();
var signed = secp256k1.ecdsaSign(signHash, Buffer.from(pkhex, "hex"));
var signatureWithoutRecid = Buffer.from(signed.signature);
var signatureWithoutRecidNormalized = secp256k1.signatureNormalize(signed.signature);
var recid = signed.recid;

// normalize to lower s
if (Buffer.compare(signatureWithoutRecid, signatureWithoutRecidNormalized) !== 0) {
    recid = recid ^ 1;
}

var recidBuffer = Buffer.alloc(1);
recidBuffer.writeUint8(recid);
var signature65 = Buffer.concat([signatureWithoutRecidNormalized, recidBuffer]);

// post transaction request to warthog node
var postdata = {
    type: 'wartTransfer',
    pinHeight: pinHeight,
    nonceId: nonceId,
    toAddr: toAddr,
    wartE8: wartE8,
    feeE8: feeE8,
    signature65: signature65.toString("hex"),
};

var res = request("POST", baseurl + "/transaction/add", { json: postdata }).body.toString();
console.log("send result:", res);
```
+++ Elixir
```elixir
baseurl = "http://localhost:3000"

# get pinHash and pinHeight from warthog node
head = HTTPoison.get!(baseurl <> "/chain/head").body |> Jason.decode!()
# the V1 API (mainnet) returns the pin fields at the top level. the V2 API
# (DeFi node) nests them: head["data"]["chainHead"]["pinHash"] and ["pinHeight"]
pinHash = head["data"]["pinHash"]
pinHeight = head["data"]["pinHeight"]

# send parameters
nonceId = 0  # 32 bit number, unique per pinHash and pinHeight
toAddr = "0000000000000000000000000000000000000000de47c9b2"  # burn destination address
wartE8 = 100000000  # 1 WART, this must be an integer, coin amount * 10^8

# round fee from WART amount
rawFee = "0.00009999"  # WARNING: NO SCIENTIFIC NOTATION
result = HTTPoison.get!(baseurl <> "/tools/encode16bit/from_string/#{rawFee}").body
encode16bit_result = Jason.decode!(result)
feeE8 = encode16bit_result["data"]["rounded"]["E8"]

# generate bytes to sign
to_sign =
    Base.decode16!(pinHash, case: :lower) <>
    <<pinHeight::big-unsigned-size(32)>> <>
    <<nonceId::big-unsigned-size(32)>> <>
    <<0::big-unsigned-size(24)>> <>
    <<feeE8::big-unsigned-size(64)>> <>
    binary_part(toAddr |> Base.decode16!(case: :lower), 0, 20) <>
    <<wartE8::big-unsigned-size(64)>>

# create normalized signature
<<recid::size(8), rs::binary-size(64)>> = Curvy.sign(to_sign, pk, hash: :sha256, compact: true, normalize: true)
signature65 = rs <> <<(recid - 31)::8>>

# post transaction request to warthog node
postdata = %{
    type: "wartTransfer",
    pinHeight: pinHeight,
    nonceId: nonceId,
    toAddr: toAddr,
    wartE8: wartE8,
    feeE8: feeE8,
    signature65: signature65 |> Base.encode16
}
HTTPoison.post!(baseurl <> "/transaction/add", Jason.encode!(postdata)).body
```
+++

## Token Transfer Transaction

Transfer a specific asset or its pool's liquidity token to another address.

+++ Python
```python
##############################
# Token Transfer Transaction
##############################

# pinHash and pinHeight come from /chain/head, see the WART Transfer example.
# the V1 API (mainnet) returns them at the top level. the V2 API
# (DeFi node) nests them: head["data"]["chainHead"]["pinHash"] and ["pinHeight"]
# send parameters
nonceId = 1  # 32 bit number, unique per pinHash and pinHeight
isLiquidity = False  # False for asset, True for liquidity tokens
assetHash = 'f45b113119c7f7c000234f1090d5d181ab60b8b24526f1edd2f563aa1ca329f2'
toAddr = '0000000000000000000000000000000000000000de47c9b2'  # burn destination address
amountU64 = 999912  # amount in smallest units

# generate bytes to sign
to_sign = (
    bytes.fromhex(pinHash) +
    pinHeight.to_bytes(4, byteorder='big') +
    nonceId.to_bytes(4, byteorder='big') +
    b'\x00\x00\x00' +
    feeE8.to_bytes(8, byteorder='big') +
    bytes.fromhex(assetHash) +
    (1 if isLiquidity else 0).to_bytes(1, byteorder='big') +
    bytes.fromhex(toAddr)[0:20] +
    amountU64.to_bytes(8, byteorder='big')
)

# sign with recovery id, RFC 6979 deterministic. sign_recoverable returns the
# 65 bytes r || s || recid, and libsecp256k1 already normalizes to low-s.
digest = sha256(to_sign).digest()
signature65 = PrivateKey(private_key).sign_recoverable(digest, hasher=None)

# post transaction request to warthog node
postdata = {
    "type": "tokenTransfer",
    "pinHeight": pinHeight,
    "nonceId": nonceId,
    "toAddr": toAddr,
    "amountU64": amountU64,
    "assetHash": assetHash,
    "isLiquidity": isLiquidity,
    "feeE8": feeE8,
    "signature65": signature65.hex()
}
rep = requests.post(baseurl + "/transaction/add", json=postdata)
print(rep.content)
```
+++ Node JS
```javascript
//////////////////////////////
// Token Transfer Transaction
//////////////////////////////

// pinHash and pinHeight come from /chain/head, see the WART Transfer example.
// the V1 API (mainnet) returns them at the top level. the V2 API
// (DeFi node) nests them: head.data.chainHead.pinHeight and head.data.chainHead.pinHash
// send parameters
var nonceId = 1;  // 32 bit number, unique per pinHash and pinHeight
var isLiquidity = false;  // false for asset, true for liquidity tokens
var assetHash = "f45b113119c7f7c000234f1090d5d181ab60b8b24526f1edd2f563aa1ca329f2"
var toAddr = "0000000000000000000000000000000000000000de47c9b2";  // burn destination address
var amountU64 = 999912;  // amount in smallest units

// generate bytes to sign
var toSign = Buffer.concat([
    Buffer.from(pinHash, "hex"),
    uint32BE(pinHeight),
    uint32BE(nonceId),
    Buffer.alloc(3),
    uint64BE(feeE8),
    Buffer.from(assetHash, "hex"),
    Buffer.from([isLiquidity ? 1 : 0]),
    Buffer.from(toAddr.slice(0, 40), "hex"),
    uint64BE(amountU64)
]);

// sign with recovery id
var signHash = createHash("sha256").update(toSign).digest();
var signed = secp256k1.ecdsaSign(signHash, Buffer.from(pkhex, "hex"));
var signatureWithoutRecid = Buffer.from(signed.signature);
var signatureWithoutRecidNormalized = secp256k1.signatureNormalize(signed.signature);
var recid = signed.recid;
if (Buffer.compare(signatureWithoutRecid, signatureWithoutRecidNormalized) !== 0) {
    recid = recid ^ 1;
}
var recidBuffer = Buffer.alloc(1);
recidBuffer.writeUint8(recid);
var signature65 = Buffer.concat([signatureWithoutRecidNormalized, recidBuffer]);

var res = request("POST", baseurl + "/transaction/add", {
    json: {
        type: 'tokenTransfer',
        pinHeight: pinHeight,
        nonceId: nonceId,
        toAddr: toAddr,
        amountU64: amountU64,
        assetHash: assetHash,
        isLiquidity: isLiquidity,
        feeE8: feeE8,
        signature65: signature65.toString("hex"),
    },
}).body.toString();
console.log("send result:", res);
```
+++ Elixir
```elixir
##############################
# Token Transfer Transaction
##############################

# pinHash and pinHeight come from /chain/head, see the WART Transfer example.
# the V1 API (mainnet) returns them at the top level. the V2 API
# (DeFi node) nests them: head["data"]["chainHead"]["pinHash"] and ["pinHeight"]
# send parameters
nonceId = 1  # 32 bit number, unique per pinHash and pinHeight
isLiquidity = false  # false for asset, true for liquidity tokens
assetHash = "f45b113119c7f7c000234f1090d5d181ab60b8b24526f1edd2f563aa1ca329f2"
toAddr = "0000000000000000000000000000000000000000de47c9b2"  # burn destination address
amountU64 = 999912  # amount in smallest units

# generate bytes to sign
to_sign =
    Base.decode16!(pinHash, case: :lower) <>
    <<pinHeight::big-unsigned-size(32)>> <>
    <<nonceId::big-unsigned-size(32)>> <>
    <<0::big-unsigned-size(24)>> <>
    <<feeE8::big-unsigned-size(64)>> <>
    Base.decode16!(assetHash, case: :lower) <>
    <<(if isLiquidity, do: 1, else: 0)::8>> <>
    binary_part(toAddr |> Base.decode16!(case: :lower), 0, 20) <>
    <<amountU64::big-unsigned-size(64)>>

# create normalized signature
<<recid::size(8), rs::binary-size(64)>> = Curvy.sign(to_sign, pk, hash: :sha256, compact: true, normalize: true)
signature65 = rs <> <<(recid - 31)::8>>

# post transaction request to warthog node
postdata = %{
    type: "tokenTransfer",
    pinHeight: pinHeight,
    nonceId: nonceId,
    toAddr: toAddr,
    amountU64: amountU64,
    assetHash: assetHash,
    isLiquidity: isLiquidity,
    feeE8: feeE8,
    signature65: signature65 |> Base.encode16
}
HTTPoison.post!(baseurl <> "/transaction/add", Jason.encode!(postdata)).body
```
+++

## Asset Creation Transaction

Create a new asset with a name, precision, and total supply.

+++ Python
```python
##############################
# Asset Creation Transaction
##############################

# pinHash and pinHeight come from /chain/head, see the WART Transfer example.
# the V1 API (mainnet) returns them at the top level. the V2 API
# (DeFi node) nests them: head["data"]["chainHead"]["pinHash"] and ["pinHeight"]
# send parameters
nonceId = 2  # 32 bit number, unique per pinHash and pinHeight
name = "TOK2"  # must be at most 5 characters
precision = 4  # 0-18, one unit = 10^precision smallest units
supplyU64 = 1000000000000  # total supply in smallest units

# generate bytes to sign (name is null-padded to 5 bytes)
nameBytes = name.encode('ascii').ljust(5, b'\x00')
to_sign = (
    bytes.fromhex(pinHash) +
    pinHeight.to_bytes(4, byteorder='big') +
    nonceId.to_bytes(4, byteorder='big') +
    b'\x00\x00\x00' +
    feeE8.to_bytes(8, byteorder='big') +
    supplyU64.to_bytes(8, byteorder='big') +
    precision.to_bytes(1, byteorder='big') +
    nameBytes
)

# sign with recovery id, RFC 6979 deterministic. sign_recoverable returns the
# 65 bytes r || s || recid, and libsecp256k1 already normalizes to low-s.
digest = sha256(to_sign).digest()
signature65 = PrivateKey(private_key).sign_recoverable(digest, hasher=None)

# post transaction request to warthog node
postdata = {
    "type": "assetCreation",
    "pinHeight": pinHeight,
    "nonceId": nonceId,
    "name": name,
    "precision": precision,
    "supplyU64": supplyU64,
    "feeE8": feeE8,
    "signature65": signature65.hex()
}
rep = requests.post(baseurl + "/transaction/add", json=postdata)
print(rep.content)
```
+++ Node JS
```javascript
//////////////////////////////
// Asset Creation Transaction
//////////////////////////////

// pinHash and pinHeight come from /chain/head, see the WART Transfer example.
// the V1 API (mainnet) returns them at the top level. the V2 API
// (DeFi node) nests them: head.data.chainHead.pinHeight and head.data.chainHead.pinHash
// send parameters
var nonceId = 2;  // 32 bit number, unique per pinHash and pinHeight
var name = "TOK2";  // must be at most 5 characters
var precision = 4;  // 0-18, one unit = 10^precision smallest units
var supplyU64 = 1000000000000;  // total supply in smallest units

// generate bytes to sign (name is null-padded to 5 bytes)
var nameBuf = Buffer.from(name, 'ascii');
var namePadded = Buffer.concat([nameBuf, Buffer.alloc(5 - name.length)]);
var toSign = Buffer.concat([
    Buffer.from(pinHash, "hex"),
    uint32BE(pinHeight),
    uint32BE(nonceId),
    Buffer.alloc(3),
    uint64BE(feeE8),
    uint64BE(supplyU64),
    Buffer.from([precision]),
    namePadded
]);

// sign with recovery id
var signHash = createHash("sha256").update(toSign).digest();
var signed = secp256k1.ecdsaSign(signHash, Buffer.from(pkhex, "hex"));
var signatureWithoutRecid = Buffer.from(signed.signature);
var signatureWithoutRecidNormalized = secp256k1.signatureNormalize(signed.signature);
var recid = signed.recid;
if (Buffer.compare(signatureWithoutRecid, signatureWithoutRecidNormalized) !== 0) {
    recid = recid ^ 1;
}
var recidBuffer = Buffer.alloc(1);
recidBuffer.writeUint8(recid);
var signature65 = Buffer.concat([signatureWithoutRecidNormalized, recidBuffer]);

var postdata = {
    type: 'assetCreation',
    name: name,
    precision: precision,
    supplyU64: supplyU64,
    pinHeight: pinHeight,
    nonceId: nonceId,
    feeE8: feeE8,
    signature65: signature65.toString("hex"),
};

var res = request("POST", baseurl + "/transaction/add", { json: postdata }).body.toString();
console.log("send result:", res);
```
+++ Elixir
```elixir
##############################
# Asset Creation Transaction
##############################

# pinHash and pinHeight come from /chain/head, see the WART Transfer example.
# the V1 API (mainnet) returns them at the top level. the V2 API
# (DeFi node) nests them: head["data"]["chainHead"]["pinHash"] and ["pinHeight"]
# send parameters
nonceId = 2  # 32 bit number, unique per pinHash and pinHeight
name = "TOK2"  # must be at most 5 characters
precision = 4  # 0-18, one unit = 10^precision smallest units
supplyU64 = 1000000000000  # total supply in smallest units

# generate bytes to sign (name is null-padded to 5 bytes)
name_padded = name |> String.pad_trailing(5, <<0>>)
to_sign =
    Base.decode16!(pinHash, case: :lower) <>
    <<pinHeight::big-unsigned-size(32)>> <>
    <<nonceId::big-unsigned-size(32)>> <>
    <<0::big-unsigned-size(24)>> <>
    <<feeE8::big-unsigned-size(64)>> <>
    <<supplyU64::big-unsigned-size(64)>> <>
    <<precision::8>> <>
    name_padded

# create normalized signature
<<recid::size(8), rs::binary-size(64)>> = Curvy.sign(to_sign, pk, hash: :sha256, compact: true, normalize: true)
signature65 = rs <> <<(recid - 31)::8>>

# post transaction request to warthog node
postdata = %{
    type: "assetCreation",
    pinHeight: pinHeight,
    nonceId: nonceId,
    name: name,
    precision: precision,
    supplyU64: supplyU64,
    feeE8: feeE8,
    signature65: signature65 |> Base.encode16
}
HTTPoison.post!(baseurl <> "/transaction/add", Jason.encode!(postdata)).body
```
+++

## Liquidity Deposit Transaction

Deposit liquidity into an asset's WART pool by providing both the asset and WART.

+++ Python
```python
##############################
# Liquidity Deposit Transaction
##############################

# pinHash and pinHeight come from /chain/head, see the WART Transfer example.
# the V1 API (mainnet) returns them at the top level. the V2 API
# (DeFi node) nests them: head["data"]["chainHead"]["pinHash"] and ["pinHeight"]
# send parameters
nonceId = 3  # 32 bit number, unique per pinHash and pinHeight
assetHash = 'f45b113119c7f7c000234f1090d5d181ab60b8b24526f1edd2f563aa1ca329f2'
baseU64 = 999912  # amount of asset in smallest units
quoteU64 = 999912  # amount of WART in E8

# generate bytes to sign
to_sign = (
    bytes.fromhex(pinHash) +
    pinHeight.to_bytes(4, byteorder='big') +
    nonceId.to_bytes(4, byteorder='big') +
    b'\x00\x00\x00' +
    feeE8.to_bytes(8, byteorder='big') +
    bytes.fromhex(assetHash) +
    baseU64.to_bytes(8, byteorder='big') +
    quoteU64.to_bytes(8, byteorder='big')
)

# sign with recovery id, RFC 6979 deterministic. sign_recoverable returns the
# 65 bytes r || s || recid, and libsecp256k1 already normalizes to low-s.
digest = sha256(to_sign).digest()
signature65 = PrivateKey(private_key).sign_recoverable(digest, hasher=None)

# post transaction request to warthog node
postdata = {
    "type": "liquidityDeposit",
    "pinHeight": pinHeight,
    "nonceId": nonceId,
    "assetHash": assetHash,
    "amountU64": baseU64,
    "wartE8": quoteU64,
    "feeE8": feeE8,
    "signature65": signature65.hex()
}
rep = requests.post(baseurl + "/transaction/add", json=postdata)
print(rep.content)
```
+++ Node JS
```javascript
//////////////////////////////
// Liquidity Deposit Transaction
//////////////////////////////

// pinHash and pinHeight come from /chain/head, see the WART Transfer example.
// the V1 API (mainnet) returns them at the top level. the V2 API
// (DeFi node) nests them: head.data.chainHead.pinHeight and head.data.chainHead.pinHash
// send parameters
var nonceId = 3;  // 32 bit number, unique per pinHash and pinHeight
var assetHash = "f45b113119c7f7c000234f1090d5d181ab60b8b24526f1edd2f563aa1ca329f2"
var baseU64 = 999912;  // amount of asset in smallest units
var quoteU64 = 999912;  // amount of WART in E8

// generate bytes to sign
var toSign = Buffer.concat([
    Buffer.from(pinHash, "hex"),
    uint32BE(pinHeight),
    uint32BE(nonceId),
    Buffer.alloc(3),
    uint64BE(feeE8),
    Buffer.from(assetHash, "hex"),
    uint64BE(baseU64),
    uint64BE(quoteU64)
]);

// sign with recovery id
var signHash = createHash("sha256").update(toSign).digest();
var signed = secp256k1.ecdsaSign(signHash, Buffer.from(pkhex, "hex"));
var signatureWithoutRecid = Buffer.from(signed.signature);
var signatureWithoutRecidNormalized = secp256k1.signatureNormalize(signed.signature);
var recid = signed.recid;
if (Buffer.compare(signatureWithoutRecid, signatureWithoutRecidNormalized) !== 0) {
    recid = recid ^ 1;
}
var recidBuffer = Buffer.alloc(1);
recidBuffer.writeUint8(recid);
var signature65 = Buffer.concat([signatureWithoutRecidNormalized, recidBuffer]);

var res = request("POST", baseurl + "/transaction/add", {
    json: {
        type: 'liquidityDeposit',
        pinHeight: pinHeight,
        nonceId: nonceId,
        assetHash: assetHash,
        amountU64: baseU64,
        wartE8: quoteU64,
        feeE8: feeE8,
        signature65: signature65.toString("hex"),
    },
}).body.toString();
console.log("send result:", res);
```
+++ Elixir
```elixir
##############################
# Liquidity Deposit Transaction
##############################

# pinHash and pinHeight come from /chain/head, see the WART Transfer example.
# the V1 API (mainnet) returns them at the top level. the V2 API
# (DeFi node) nests them: head["data"]["chainHead"]["pinHash"] and ["pinHeight"]
# send parameters
nonceId = 3  # 32 bit number, unique per pinHash and pinHeight
assetHash = "f45b113119c7f7c000234f1090d5d181ab60b8b24526f1edd2f563aa1ca329f2"
baseU64 = 999912  # amount of asset in smallest units
quoteU64 = 999912  # amount of WART in E8

# generate bytes to sign
to_sign =
    Base.decode16!(pinHash, case: :lower) <>
    <<pinHeight::big-unsigned-size(32)>> <>
    <<nonceId::big-unsigned-size(32)>> <>
    <<0::big-unsigned-size(24)>> <>
    <<feeE8::big-unsigned-size(64)>> <>
    Base.decode16!(assetHash, case: :lower) <>
    <<baseU64::big-unsigned-size(64)>> <>
    <<quoteU64::big-unsigned-size(64)>>

# create normalized signature
<<recid::size(8), rs::binary-size(64)>> = Curvy.sign(to_sign, pk, hash: :sha256, compact: true, normalize: true)
signature65 = rs <> <<(recid - 31)::8>>

# post transaction request to warthog node
postdata = %{
    type: "liquidityDeposit",
    pinHeight: pinHeight,
    nonceId: nonceId,
    assetHash: assetHash,
    amountU64: baseU64,
    wartE8: quoteU64,
    feeE8: feeE8,
    signature65: signature65 |> Base.encode16
}
HTTPoison.post!(baseurl <> "/transaction/add", Jason.encode!(postdata)).body
```
+++

## Liquidity Withdrawal Transaction

Withdraw liquidity from an asset's WART pool by redeeming liquidity tokens.

+++ Python
```python
##############################
# Liquidity Withdrawal Transaction
##############################

# pinHash and pinHeight come from /chain/head, see the WART Transfer example.
# the V1 API (mainnet) returns them at the top level. the V2 API
# (DeFi node) nests them: head["data"]["chainHead"]["pinHash"] and ["pinHeight"]
# send parameters
nonceId = 4  # 32 bit number, unique per pinHash and pinHeight
assetHash = 'f45b113119c7f7c000234f1090d5d181ab60b8b24526f1edd2f563aa1ca329f2'
sharesE8 = 100  # amount of liquidity tokens to redeem in E8

# generate bytes to sign
to_sign = (
    bytes.fromhex(pinHash) +
    pinHeight.to_bytes(4, byteorder='big') +
    nonceId.to_bytes(4, byteorder='big') +
    b'\x00\x00\x00' +
    feeE8.to_bytes(8, byteorder='big') +
    bytes.fromhex(assetHash) +
    sharesE8.to_bytes(8, byteorder='big')
)

# sign with recovery id, RFC 6979 deterministic. sign_recoverable returns the
# 65 bytes r || s || recid, and libsecp256k1 already normalizes to low-s.
digest = sha256(to_sign).digest()
signature65 = PrivateKey(private_key).sign_recoverable(digest, hasher=None)

# post transaction request to warthog node
postdata = {
    "type": "liquidityWithdrawal",
    "pinHeight": pinHeight,
    "nonceId": nonceId,
    "assetHash": assetHash,
    "amountE8": sharesE8,
    "feeE8": feeE8,
    "signature65": signature65.hex()
}
rep = requests.post(baseurl + "/transaction/add", json=postdata)
print(rep.content)
```
+++ Node JS
```javascript
//////////////////////////////
// Liquidity Withdrawal Transaction
//////////////////////////////

// pinHash and pinHeight come from /chain/head, see the WART Transfer example.
// the V1 API (mainnet) returns them at the top level. the V2 API
// (DeFi node) nests them: head.data.chainHead.pinHeight and head.data.chainHead.pinHash
// send parameters
var nonceId = 4;  // 32 bit number, unique per pinHash and pinHeight
var assetHash = "f45b113119c7f7c000234f1090d5d181ab60b8b24526f1edd2f563aa1ca329f2"
var sharesE8 = 100;  // amount of liquidity tokens to redeem in E8

// generate bytes to sign
var toSign = Buffer.concat([
    Buffer.from(pinHash, "hex"),
    uint32BE(pinHeight),
    uint32BE(nonceId),
    Buffer.alloc(3),
    uint64BE(feeE8),
    Buffer.from(assetHash, "hex"),
    uint64BE(sharesE8)
]);

// sign with recovery id
var signHash = createHash("sha256").update(toSign).digest();
var signed = secp256k1.ecdsaSign(signHash, Buffer.from(pkhex, "hex"));
var signatureWithoutRecid = Buffer.from(signed.signature);
var signatureWithoutRecidNormalized = secp256k1.signatureNormalize(signed.signature);
var recid = signed.recid;
if (Buffer.compare(signatureWithoutRecid, signatureWithoutRecidNormalized) !== 0) {
    recid = recid ^ 1;
}
var recidBuffer = Buffer.alloc(1);
recidBuffer.writeUint8(recid);
var signature65 = Buffer.concat([signatureWithoutRecidNormalized, recidBuffer]);

var res = request("POST", baseurl + "/transaction/add", {
    json: {
        type: 'liquidityWithdrawal',
        pinHeight: pinHeight,
        nonceId: nonceId,
        assetHash: assetHash,
        amountE8: sharesE8,
        feeE8: feeE8,
        signature65: signature65.toString("hex"),
    },
}).body.toString();
console.log("send result:", res);
```
+++ Elixir
```elixir
##############################
# Liquidity Withdrawal Transaction
##############################

# pinHash and pinHeight come from /chain/head, see the WART Transfer example.
# the V1 API (mainnet) returns them at the top level. the V2 API
# (DeFi node) nests them: head["data"]["chainHead"]["pinHash"] and ["pinHeight"]
# send parameters
nonceId = 4  # 32 bit number, unique per pinHash and pinHeight
assetHash = "f45b113119c7f7c000234f1090d5d181ab60b8b24526f1edd2f563aa1ca329f2"
sharesE8 = 100  # amount of liquidity tokens to redeem in E8

# generate bytes to sign
to_sign =
    Base.decode16!(pinHash, case: :lower) <>
    <<pinHeight::big-unsigned-size(32)>> <>
    <<nonceId::big-unsigned-size(32)>> <>
    <<0::big-unsigned-size(24)>> <>
    <<feeE8::big-unsigned-size(64)>> <>
    Base.decode16!(assetHash, case: :lower) <>
    <<sharesE8::big-unsigned-size(64)>>

# create normalized signature
<<recid::size(8), rs::binary-size(64)>> = Curvy.sign(to_sign, pk, hash: :sha256, compact: true, normalize: true)
signature65 = rs <> <<(recid - 31)::8>>

# post transaction request to warthog node
postdata = %{
    type: "liquidityWithdrawal",
    pinHeight: pinHeight,
    nonceId: nonceId,
    assetHash: assetHash,
    amountE8: sharesE8,
    feeE8: feeE8,
    signature65: signature65 |> Base.encode16
}
HTTPoison.post!(baseurl <> "/transaction/add", Jason.encode!(postdata)).body
```
+++

## Cancelation Transaction

Cancel a pending transaction or an existing order. You must specify the `cancelHeight` and `cancelNonceId` of the transaction to cancel.

+++ Python
```python
##############################
# Cancelation Transaction
##############################

# pinHash and pinHeight come from /chain/head, see the WART Transfer example.
# the V1 API (mainnet) returns them at the top level. the V2 API
# (DeFi node) nests them: head["data"]["chainHead"]["pinHash"] and ["pinHeight"]
# send parameters
nonceId = 5  # 32 bit number, unique per pinHash and pinHeight
cancelHeight = 0  # the pinHeight of the transaction to cancel
cancelNonceId = 1  # the nonceId of the transaction to cancel

# generate bytes to sign
to_sign = (
    bytes.fromhex(pinHash) +
    pinHeight.to_bytes(4, byteorder='big') +
    nonceId.to_bytes(4, byteorder='big') +
    b'\x00\x00\x00' +
    feeE8.to_bytes(8, byteorder='big') +
    cancelHeight.to_bytes(4, byteorder='big') +
    cancelNonceId.to_bytes(4, byteorder='big')
)

# sign with recovery id, RFC 6979 deterministic. sign_recoverable returns the
# 65 bytes r || s || recid, and libsecp256k1 already normalizes to low-s.
digest = sha256(to_sign).digest()
signature65 = PrivateKey(private_key).sign_recoverable(digest, hasher=None)

# post transaction request to warthog node
postdata = {
    "type": "cancelation",
    "pinHeight": pinHeight,
    "nonceId": nonceId,
    "cancelHeight": cancelHeight,
    "cancelNonceId": cancelNonceId,
    "feeE8": feeE8,
    "signature65": signature65.hex()
}
rep = requests.post(baseurl + "/transaction/add", json=postdata)
print(rep.content)
```
+++ Node JS
```javascript
//////////////////////////////
// Cancelation Transaction
//////////////////////////////

// pinHash and pinHeight come from /chain/head, see the WART Transfer example.
// the V1 API (mainnet) returns them at the top level. the V2 API
// (DeFi node) nests them: head.data.chainHead.pinHeight and head.data.chainHead.pinHash
// send parameters
var nonceId = 5;  // 32 bit number, unique per pinHash and pinHeight
var cancelHeight = 0;  // the pinHeight of the transaction to cancel
var cancelNonceId = 1;  // the nonceId of the transaction to cancel

// generate bytes to sign
var toSign = Buffer.concat([
    Buffer.from(pinHash, "hex"),
    uint32BE(pinHeight),
    uint32BE(nonceId),
    Buffer.alloc(3),
    uint64BE(feeE8),
    uint32BE(cancelHeight),
    uint32BE(cancelNonceId)
]);

// sign with recovery id
var signHash = createHash("sha256").update(toSign).digest();
var signed = secp256k1.ecdsaSign(signHash, Buffer.from(pkhex, "hex"));
var signatureWithoutRecid = Buffer.from(signed.signature);
var signatureWithoutRecidNormalized = secp256k1.signatureNormalize(signed.signature);
var recid = signed.recid;
if (Buffer.compare(signatureWithoutRecid, signatureWithoutRecidNormalized) !== 0) {
    recid = recid ^ 1;
}
var recidBuffer = Buffer.alloc(1);
recidBuffer.writeUint8(recid);
var signature65 = Buffer.concat([signatureWithoutRecidNormalized, recidBuffer]);

var res = request("POST", baseurl + "/transaction/add", {
    json: {
        type: 'cancelation',
        pinHeight: pinHeight,
        nonceId: nonceId,
        cancelHeight: cancelHeight,
        cancelNonceId: cancelNonceId,
        feeE8: feeE8,
        signature65: signature65.toString("hex"),
    },
}).body.toString();
console.log("send result:", res);
```
+++ Elixir
```elixir
##############################
# Cancelation Transaction
##############################

# pinHash and pinHeight come from /chain/head, see the WART Transfer example.
# the V1 API (mainnet) returns them at the top level. the V2 API
# (DeFi node) nests them: head["data"]["chainHead"]["pinHash"] and ["pinHeight"]
# send parameters
nonceId = 5  # 32 bit number, unique per pinHash and pinHeight
cancelHeight = 0  # the pinHeight of the transaction to cancel
cancelNonceId = 1  # the nonceId of the transaction to cancel

# generate bytes to sign
to_sign =
    Base.decode16!(pinHash, case: :lower) <>
    <<pinHeight::big-unsigned-size(32)>> <>
    <<nonceId::big-unsigned-size(32)>> <>
    <<0::big-unsigned-size(24)>> <>
    <<feeE8::big-unsigned-size(64)>> <>
    <<cancelHeight::big-unsigned-size(32)>> <>
    <<cancelNonceId::big-unsigned-size(32)>>

# create normalized signature
<<recid::size(8), rs::binary-size(64)>> = Curvy.sign(to_sign, pk, hash: :sha256, compact: true, normalize: true)
signature65 = rs <> <<(recid - 31)::8>>

# post transaction request to warthog node
postdata = %{
    type: "cancelation",
    pinHeight: pinHeight,
    nonceId: nonceId,
    cancelHeight: cancelHeight,
    cancelNonceId: cancelNonceId,
    feeE8: feeE8,
    signature65: signature65 |> Base.encode16
}
HTTPoison.post!(baseurl <> "/transaction/add", Jason.encode!(postdata)).body
```
+++

## Transaction Reference

For detailed information on all transaction types, signature byte layouts, and parameter requirements, see the [Transaction API documentation](../api/rest/create-transaction.md).
