# Miner Integration

## Condition to Mine a Block
[!ref](/Janushash/condition-to-solve-a-block.md)

## Mining Strategy
[!ref](/Janushash/mining.md)

## Stratum Protocol
[!ref](Pools/stratum.md)

## Reporting Hashrate
Miners should report Sha256t and Verushash v2.1 measured hashrates and in addition to that they should report the Janusscore. Janusscore describes the mining efficiency (combined hashrate) and can be computed from `sha256t` and `verushash` hashrate variables as follows:
```c++
(10.0 / 3.0) * double(sha256t) * (pow(c + double(verus) / double(sha256t), 0.3) - pow(c, 0.3));
```
[!ref](/janushash/analysis/janusscore.md)

## Pools for testing
[!ref](/links.md)
