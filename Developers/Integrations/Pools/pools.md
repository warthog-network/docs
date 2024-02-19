# Pool Integration

## Share validation

I have written a C++ backend which support computes the hash product

- Verushash(header)*Sha256t(header)^0.7

This is a number between 0 and 1. If the first condition 

1. Sha256t(header) > c for some hard-coded constant c.

is not met it returns 1. This is a convenient way for pools to check valid shares: Instead of using the block real target, the pool would set an easier difficulty in Stratum jobs, which corresponds to a larger target $t^*$ than the block target $t$. A share is valid if the hash product returned by the backend is smaller than $t^*$. If the returned value is 1 as is done by the backend if the first condition above is not met, the share is automatically invalid because $t^*$ will be between 0 and 1.



[The pool backend](https://github.com/warthog-network/pool_backend) has [two endpoints](https://github.com/warthog-network/pool_backend/blob/master/backend.cpp#L63-L85):

1. `/score/:headerhex`: It computes the score above (hash product). For performance the score is not embedded into json encoded but directly returned. On error an empty string is returned. `headerhex` is the hex encoded header (80 bytes = 160 bytes hex encoded).
2. `/target_to_double/:targethex`: It converts the block target from byte representation to double. This is important to check if the score actually solves a block.


## Block structure
Each block consists of a 10 extranonce bytes followed by 3 sections:

#### 1. New  address section (`2+n1*20` bytes)
This section starts by encoding the number `n1` of new addresses registered in this block, then `n1` elements of `20 ` bytes each follow.

#### 2. Reward section (`16` bytes)
This section encodes the mining reward (miner + amount).

#### 3. Transfer section (`4+n3*99` bytes)
It starts by encoding the number `n3` of transfers in a 4 byte network byte order integer. Then follow `n3` entries of 99 bytes each, one for each transfer. If `n3` is 0 then the transfer section is omitted completely, in this case it is 0 bytes long. Miner devs can detect this by comparing the current cursor with the end of the block byte vector when parsing the sections of a block.


## Merkle Root Computation
Merkle tree has the `n1 + 1 + n3` elements of the three sections as its leaves: `n1` 20 byte elements, one 16 byte element and `n3` 99 byte elements. The Merkle tree uses `sha256` to combine two elements into one in each hierarchy level.

Merkle root is computed this way:

```c++
Hash BodyView::merkleRoot() const
{
    std::vector<Hash> hashes(nAddresses + nRewards + nTransfers);

    // hash addresses
    size_t idx = 0;
    for (size_t i = 0; i < nAddresses; ++i)
        hashes[idx++] = hashSHA256(s.data() + offsetAddresses + i * AddressSize, AddressSize);

    // hash payouts
    for (size_t i = 0; i < nRewards; ++i)
        hashes[idx++] = hashSHA256(data() + offsetRewards + i * RewardSize, RewardSize);
    // hash payments
    for (size_t i = 0; i < nTransfers; ++i)
        hashes[idx++] = hashSHA256(data() + offsetTransfers + i * TransferSize, TransferSize);
    std::vector<Hash> tmp, *from, *to;
    from = &hashes;
    to = &tmp;

    do {
        to->resize((from->size() + 1) / 2);
        size_t j = 0;
        for (size_t i = 0; i < (from->size() + 1) / 2; ++i) {
            HasherSHA256 hasher {};
            hasher.write((*from)[j].data(), 32);
            if (j + 1 < from->size()) {
                hasher.write((*from)[j + 1].data(), 32);
            }

            if (to->size() == 1)
                hasher.write(data(), 10);
            (*to)[i] = std::move(hasher);
            j += 2;
        }
        std::swap(from, to);
    } while (from->size() > 1);
    return from->front();
}
```

In the last step we append the 10 byte extranonce space from the beginning of the block to the hashed data for the final hash. The data we append these 10 `extranonce` to we call `merklePrefix`. The Merkle root is therefore `sha256(merklePrefix + extranonceBytes)`. The `merklePrefix` is transmitted in the Stratum protocol. It is either 32 bytes or 64 bytes long. Only if you have 1 leaf in the Merkle tree, the Merkle prefix will be 32 bytes, otherwise the Merkle prefix will be the two children of the root concatenated.

## Scanning the Chain
[!ref](../../api.md)

## Pool Payout
[!ref](../wallet-integration.md)

## Miners for Testing
[!ref](/links.md)
