# Stratum Protocol
Messages are newline (`"\n"`) terminated json strings sent over plain TCP with optional TLS encryption. URI scheme is `stratum+tcp` for plain TCP and `stratum+ssl` or `stratum+tls` if TLS encryption is used (for some reason "`ssl`" is more prevalent than "`tls`" as URI scheme despite in fact TLS is actually used). 


## Establishing a stratum connection
After establishing a TCP connection (and optionally TLS) the first message is sent from the Client and must be of type `mining.subscribe`:
### Mining.subscribe
#### Request from client:
```
{
  "id": 1,
  "method": "mining.subscribe",
  "params": [
    "MinerName/1.0.0",           # user agent / version
    "<suggested session id>"     # optional suggested session Id
  ]
}

```
#### Response from pool:
```
{
  "id": 1,                 # same as "id" in the request
  "result": [
    [
      [
        "mining.notify",
        "extra1hex"        # session ID, often just equal to the extra nonce 1
      ]
    ],
    "extra1hex",           # extranonce 1 in hex
    2                      # size of extranonce2
  ],
  "error": null
}
```

### Mining authorize
The next message is again send from the client and must have `"method"`  field equal to `"mining.authorize"`:
#### Request from client:
```
{
  "id": 2, 
  "method": "mining.authorize",
  "params": [
    "address.worker",  # user name, convention is to specify address and worker id
    "password"         # password
  ]
}
```

#### Response from pool:
```
{
  "id": 2,        # same "id" as in the request
  "result": true,
  "error": null
}
```

Now the connection is established. The pool n
## Events pushed from pool server to clients
There are two kind of events that sent from the pool: `"mining.notify"` and `"mining.set_difficulty"`.

### Mining.notify
```
{
  "id": null,
  "method": "mining.notify",
  "params": [
    "jobId",         # jobId has to be sent back on submission
    "prevHash",      # hex encoded previous hash in block header
    "merklePrefix",  # hex encoded prefix of 
    "version",       # hex encoded block version
    "nbits",         # hex encoded nbits, difficulty target
    "ntime",         # hex encoded ntime, timestamp in block header
    false            # clean? If yes, discard old jobs.
  ]
}
```
#### Notable differences from Bitcoin's Stratum protocol:
This method has only 7 parameters instead of 9 parameters in Bitcoin's stratum protocol. Bitcoin's stratum protocol's third (coinbase 1) and fourth (coinbase 2) parameters are not necessary in Warthog. Furthermore instead of Bitcoin's *Merkle branches* parameter we have a Merkle prefix parameter. Merkle root is computed as follows:

#### Merkle root computation
Merkle root is `sha256(merklePrefix + extranonce1 + extranonce2)`. Miners can choose extranonce2 arbitrarily (with length specified by pool) to generate different Merkle roots.

#### Header structure
The parameters sent in a `"mining.notify"` event and a 4-byte `nonce` selected by the miner can be used to form a header to be hashed by the miner as follows:

Bytes | Meaning
------|--------
1-32  | prevHash
33-36 | nbits
37-68 | Merkle root (controlled by miner, see above)
69-72 | version
73-76 | ntime
77-80 | nonce (chosen by miner)

### Mining.set_difficulty

```
{
  "id": null,
  "method": "mining.set_difficulty",
  "params": [
    100000     # difficulty
  ]
}
```

#### Notable differences from Bitcoin's Stratum protocol
In contrast to Bitcoin's stratum protocol the target is just the inverse of the difficulty. In Bitcoin there is an additional factor of 2^32 involved for historical reasons. Since Warthog was written from scratch, it not carry this historical burden.
This means the miner must meet the target `1/difficulty` to mine a share. 

## Events pushed from miner to pool
### Mining.submit
When the miner has found nonce and extranonce2 such that the block is 
```
{
  "id": 4,
  "method": "mining.submit",
  "params": [
    "jobId",       # jobId from mining.notify
    "0000",        # extranonce2 hex
    "a5b378fe",    # time hex (in shifupool equal to ntime from mining.notify)
    "a28a04a2"     # nonce hex
  ]
}
```

Pool will reply with
```
{
  "id": 4,        # same "id" as in the request
  "result": true,
  "error": null
}
```
or report error.
## Error reporting
Pool response must specify request id in the `"id"` field. Errors are reported by setting `"result"` to `null` and specifying error details in an array of size 3 (code, message, additional info) in the `"error"` field.

```
{
  "id": 3,
  "result": null,
  "error": [
    1,              # error code
    "error msg",    # error message
    null            # additional error information
  ]
}
```


01/21/2024 6:07 PM
Every coin has its own stratum specification and I changed just very little. Basically we do not have coinbases but mining.notify looks like this:

```
{
  "id": null,
  "method": "mining.notify",
  "params": [
    "jobId",         # jobId has to be sent back on submission
    "prevHash",      # previous hash in block header
    "merklePrefix",  # prefix of 
    2,               # version
    "nbits",         # nbits, difficulty target
    "ntime",         # ntime, timestamp in block header
    false            # clean? If yes, discard old jobs.
  ]
}
```
