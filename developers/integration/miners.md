---
title: Miners
---
# Miner Integration

## Condition to Mine a Block
Warthog's Janushash algorithm requires a custom condition on two hashes (SHA256t and Verushash v2.1) to be satisfied for valid blocks. 
[!ref Condition to solve a block](../../Unique-Features/Janushash/condition-to-solve-a-block.md)

## Mining Strategy
Optimally mining blocks requires a two-stage strategy. First SHA256t hashes have to be calculated (since SHA256t is faster than Verushash v2.1 on modern hardware), then promising candidates have to be filtered for evaluation of Verushash v2.1.

[!ref Mining Strategy](/Unique-Features/Janushash/mining.md)

## Stratum Protocol
[!ref](Pools/stratum.md)

## Reporting Hashrate
Miners should report Sha256t and Verushash v2.1 measured hashrates and in addition to that they should report the Janusscore. Janusscore describes the mining efficiency (combined hashrate) and can be computed from `sha256t` and `verushash` hashrate variables as follows:
```c++
(10.0 / 3.0) * double(sha256t) * (pow(c + double(verus) / double(sha256t), 0.3) - pow(c, 0.3));
```
[!ref](../../Unique Features/Janushash/analysis/janusscore.md)

## Pools for testing
[!ref](/links.md)
