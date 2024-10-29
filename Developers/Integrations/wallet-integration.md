# Wallet integration
In this guide we demonstrate how to handle wallets and send transactions in Python 3, NodeJS and Elixir. 
## Prerequisites
+++ Python
We use the `pycryptodome` and `pycoin` Python packages.
+++ Node JS
We use the `elliptic`, `node:crypto`, `sync-request` and `secp256k1` NodeJS packages.
+++ Elixir
We use the `{:curvy, "~> 0.3.1"}`, `{:jason, "~> 1.4"}` and `{:httpoison, "~> 2.0"}` Hex packages.
+++

## Handling wallets

Below we demonstrate how to
- generate a new private key,
- load a private key from hexadecimal encoding, 
- derive a public key from a private key,
- derive a raw Warthog address form a public key and
- extend the raw Warthog address by its checksum to obtain an ordinary Warthog address. 
+++ Python 
```python
from ecdsa import SigningKey, SECP256k1

##############################
# Handling wallets
##############################

# generate private key
pk = SigningKey.generate(curve=SECP256k1)

# alternatively read private key
pkhex= '966a71a98bb5d13e9116c0dffa3f1a7877e45c6f563897b96cfd5c59bf0803e0'
pk = SigningKey.from_string(bytes.fromhex(pkhex),curve=SECP256k1)

# print private key
print("private_key:")
print(pk.to_string().hex())

# derive public key
pubkey = pk.get_verifying_key().to_string('compressed')

# print public key
print("public key:")
print(pubkey.hex())

# convert public key to raw addresss
from Crypto.Hash import RIPEMD160, SHA256
sha = SHA256.SHA256Hash(pubkey).digest()
addr_raw = RIPEMD160.RIPEMD160Hash(sha).digest()
addr_raw.hex()

# generate address by appending checksum
addr_hash = SHA256.SHA256Hash(addr_raw).digest()
checksum = addr_hash[0:4]
addr = addr_raw + checksum

# print address
print("address:")
print(addr.hex())
```
+++ Node JS
```javascript
const elliptic = require('elliptic');
const crypto = require('node:crypto');
const ec = new elliptic.ec('secp256k1');

//////////////////////////////
// Handling wallets
//////////////////////////////


// generate private key
var pk = ec.genKeyPair()

// alternatively read private key
var pkhex= '966a71a98bb5d13e9116c0dffa3f1a7877e45c6f563897b96cfd5c59bf0803e0'
var pk =  ec.keyFromPrivate(pkhex);

// convert private key to hex
pkhex = pk.getPrivate().toString("hex")
while (pkhex.length < 64) {
  pkhex = "0" + pkhex;
}

// print private key:
console.log("private key:", pkhex)

// derive public key
var pubKey = pk.getPublic().encodeCompressed("hex");

// print public key
console.log("public key:", pubKey)

// convert public key to raw addresss
var sha = crypto.createHash('sha256').update(Buffer.from(pubKey,"hex")).digest()
var addrRaw = crypto.createHash('ripemd160').update(sha).digest()

// generate address by appending checksum
var checksum = crypto.createHash('sha256').update(addrRaw).digest().slice(0,4)
var addr = Buffer.concat([addrRaw , checksum]).toString("hex")

// print address
console.log("address:", addr)

```
+++ Elixir
```elixir
##############################
# Handling wallets
##############################
#
# generate private key
pk = Curvy.generate_key()

# alternatively read private key
pk = "966a71a98bb5d13e9116c0dffa3f1a7877e45c6f563897b96cfd5c59bf0803e0" |> Base.decode16!(case: :lower) |> Curvy.Key.from_privkey()

# print private key
Curvy.Key.to_privkey(pk)|>Base.encode16|>IO.puts

# derive public key
pubkey = Curvy.Key.to_pubkey(pk)

# print public key
pubkey|>Base.encode16|>IO.puts

# convert public key to raw addresss
addr_raw = :crypto.hash(:ripemd160,:crypto.hash(:sha256,pubkey))

# generate address by appending checksum
<< checksum :: binary-size(4),_::binary>> =  :crypto.hash(:sha256,addr_raw)
addr= addr_raw<>checksum

# print address
IO.puts("address: #{addr|>Base.encode16()}")
```
+++

## Generating, signing and sending a transfer transaction

In Warthog transactions are pinned to a specific block the blockchain. The height of this block is its `pinHeight`, its hash the `pinHash`. This information has to be retrieved from the node in order to start generating a transaction, a transaction with a `pinHeight` that is too old be discarded. 
This serves two purposes
1. Users can be sure that transactions will not be pending forever.
2. By choosing a particular `pinHeight` users can actively control the time to live (TTL) of a pending transaction before it is discarded.

Not all height values are valid `pinHeights`, instead they must be a multiple of 32. For convenience the `/chain/head` endpoint provides the latest `pinHeight` value together with the corresponding block hash `pinHash`. In our code sample below we will use this endpoint.

In addition we demonstrate how to create the bytes to sign, how a valid sekp256k1 recoverable signature in custom format is generated and how to send the transaction to the Warthog node via a HTTP POST request. For reference read the [API guide](API.md). The below code uses the private key `pk` variable from the above code snippet.

+++ Python
```python
##############################
# Generate a signed transaction
##############################
import requests
import json
baseurl="http://localhost:3000"

# get pinHash and pinHeight from warthog node
head_raw = requests.get(baseurl+'/chain/head').content
head = json.loads(head_raw)
pinHash =  head["data"]["pinHash"]
pinHeight = head["data"]["pinHeight"]


# send parameters
nonceId = 0 # 32 bit number, unique per pinHash and pinHeight
toAddr = '0000000000000000000000000000000000000000de47c9b2' # burn destination address
amountE8 = 100000000 # 1 WART, this must be an integer, coin amount * 10E8


# round fee from WART amount
rawFee = "0.00009999" # this needs to be rounded, WARNING: NO SCIENTIFIC NOTATION
result = requests.get(baseurl+'/tools/encode16bit/from_string/'+rawFee).content
encode16bit_result = json.loads(result)
feeE8 = encode16bit_result["data"]["roundedE8"] # 9992


# alternative: round fee from E8 amount
rawFeeE8 = "9999" # this needs to be rounded
result = requests.get(baseurl+'/tools/encode16bit/from_e8/'+rawFeeE8).content
encode16bit_result = json.loads(result)
feeE8 = encode16bit_result["data"]["roundedE8"] # 9992


# generate bytes to sign
to_sign =\
bytes.fromhex(pinHash)+\
pinHeight.to_bytes(4, byteorder='big') +\
nonceId.to_bytes(4, byteorder='big') +\
b'\x00\x00\x00'+\
feeE8.to_bytes(8, byteorder='big')+\
bytes.fromhex(toAddr)[0:20]+\
amountE8.to_bytes(8, byteorder='big')


# create signature
from pycoin.ecdsa.secp256k1 import secp256k1_generator
from hashlib import sha256
private_key = pk.privkey.secret_multiplier 
digest = sha256(to_sign).digest()


# sign with recovery id
(r, s, rec_id) = secp256k1_generator.sign_with_recid(private_key, int.from_bytes(digest, 'big'))


# normalize to lower s
if s > secp256k1_generator.order()/2: #
    s = secp256k1_generator.order() - s
    rec_id ^= 1 # https://github.com/bitcoin-core/secp256k1/blob/e72103932d5421f3ae501f4eba5452b1b454cb6e/src/ecdsa_impl.h#L295
signature65 = r.to_bytes(32,byteorder='big')+s.to_bytes(32,byteorder='big')+rec_id.to_bytes(1,byteorder='big')


# post transaction request to warthog node
postdata = {
 "pinHeight": pinHeight,
 "nonceId": nonceId,
 "toAddr": toAddr,
 "amountE8": amountE8,
 "feeE8": feeE8,
 "signature65": signature65.hex()
}
rep = requests.post(baseurl + "/transaction/add", json = postdata)
rep.content
```
+++ Node JS
!!!
Javascript does not support 64 bit integers natively, one has to resort to `BigInt`, which does not serialize as a JSON number required by the API. Therefore we are just serializing a normal Javascript number which will only be accurate up to 15 digits, i.e. only 7 figure WART amounts can be precisely handled this way because Warthog has a precision of 8 digits after comma. The total hard cap of WART is 18m which is a 8 figure number, so practically the 7 figures should be sufficient for most purposes.
!!!


```javascript
//////////////////////////////
// Generate a signed transaction
//////////////////////////////
var crypto = require("crypto");
var request = require("sync-request");
var secp256k1 = require("secp256k1");

var baseurl = "http://localhost:3000";

// get pinHash and pinHeight from warthog node
var head = JSON.parse(request("GET", baseurl + "/chain/head").body.toString());
var pinHeight = head.data.pinHeight;
var pinHash = head.data.pinHash;

// send parameters
var nonceId = 0; // 32 bit number, unique per pinHash and pinHeight
var toAddr = "0000000000000000000000000000000000000000de47c9b2"; // burn destination address
var amountE8 = 100000000; // 1 WART, this must be an integer, coin amount * 10E8

// round fee from WART amount
var rawFee = "0.00009999"; // this needs to be rounded, WARNING: NO SCIENTIFIC NOTATION
var result = request(
  "GET",
  baseurl + "/tools/encode16bit/from_string/" + rawFee
).body.toString();
var encode16bit_result = JSON.parse(result);
var feeE8 = encode16bit_result["data"]["roundedE8"]; // 9992

// alternative: round fee from E8 amount
var rawFeeE8 = "9999"; // this needs to be rounded
result = request(
  "GET",
  baseurl + "/tools/encode16bit/from_e8/" + rawFeeE8
).body.toString();
encode16bit_result = JSON.parse(result);
feeE8 = encode16bit_result["data"]["roundedE8"]; // 9992

// generate bytes to sign
var buf1 = Buffer.from(pinHash, "hex");
var buf2 = Buffer.alloc(19);
buf2.writeUInt32BE(pinHeight, 0);
buf2.writeUInt32BE(nonceId, 4);
buf2.writeUInt8(0, 8);
buf2.writeUInt8(0, 9);
buf2.writeUInt8(0, 10);
buf2.writeBigUInt64BE(BigInt(feeE8), 11);
var buf3 = Buffer.from(toAddr.slice(0, 40), "hex");
var buf4 = Buffer.alloc(8);
buf4.writeBigUInt64BE(BigInt(amountE8), 0);
var toSign = Buffer.concat([buf1, buf2, buf3, buf4]);

// sign with recovery id
var signHash = crypto.createHash("sha256").update(toSign).digest();
var signed = secp256k1.ecdsaSign(signHash, Buffer.from(pkhex, "hex"));
var signatureWithoutRecid = Buffer.from(signed.signature);
var signatureWithoutRecidNormalized = secp256k1.signatureNormalize(
  signed.signature
);
var recid = signed.recid;

// normalize to lower s
if (
  Buffer.compare(signatureWithoutRecid, signatureWithoutRecidNormalized) !== 0
) {
  recid = recid ^ 1;
}

var recidBuffer = Buffer.alloc(1);
recidBuffer.writeUint8(recid);

// form full signature
var signature65 = Buffer.concat([signatureWithoutRecidNormalized, recidBuffer]);

// post transaction request to warthog node
var postdata = {
  pinHeight: pinHeight,
  nonceId: nonceId,
  toAddr: toAddr,
  amountE8: amountE8,
  feeE8: feeE8,
  signature65: signature65.toString("hex"),
};

var res = request("POST", baseurl + "/transaction/add", {
  json: postdata,
}).body.toString();
console.log("send result:", res);
```
+++ Elixir
```elixir
##############################
# Generate a signed transaction
##############################
baseurl = "http://localhost:3000"


# get pinHash and pinHeight from warthog node
head = HTTPoison.get!(baseurl <> "/chain/head").body|>Jason.decode!()
pinHash =  head["data"]["pinHash"]
pinHeight = head["data"]["pinHeight"]


# send parameters
nonceId = 0 # 32 bit number, unique per pinHash and pinHeight
toAddr = "0000000000000000000000000000000000000000de47c9b2" # burn destination address
amountE8 = 100000000 # 1 WART, this must be an integer, coin amount * 10E8


# round fee from WART amount
rawFee = "0.00009999" # this needs to be rounded, WARNING: NO SCIENTIFIC NOTATION
result = HTTPoison.get!(baseurl<>"/tools/encode16bit/from_string/#{rawFee}").body
encode16bit_result = Jason.decode!(result)
feeE8 = encode16bit_result["data"]["roundedE8"] # 9992


# alternative: round fee from E8 amount
rawFeeE8 = "9999" # this needs to be rounded
result = HTTPoison.get!(baseurl<>"/tools/encode16bit/from_e8/#{rawFeeE8}").body
encode16bit_result = Jason.decode!(result)
feeE8 = encode16bit_result["data"]["roundedE8"] # 9992


# generate bytes to sign
to_sign = 
  Base.decode16!(pinHash, case: :lower) <> 
  <<pinHeight ::big-unsigned-size(32)>> <>
  <<nonceId ::big-unsigned-size(32)>> <>
  <<0 ::big-unsigned-size(24)>> <>
  <<feeE8 ::big-unsigned-size(64)>> <>
    binary_part(toAddr|>Base.decode16!(case: :lower),0,20) <>
  <<amountE8 ::big-unsigned-size(64)>>

# create normalized signature
<<recid::size(8), rs::binary-size(64)>> = Curvy.sign(to_sign, pk, hash: :sha256, compact: true, normalize: true)
signature65 = rs <> <<(recid-31) ::8>> 


# post transaction request to warthog node
postdata = %{
 pinHeight: pinHeight,
 nonceId: nonceId,
 toAddr: toAddr,
 amountE8: amountE8,
 feeE8: feeE8,
 signature65: signature65 |> Base.encode16
}
HTTPoison.post!(baseurl <> "/transaction/add", Jason.encode!(postdata)).body
```
+++
