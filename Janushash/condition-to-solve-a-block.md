---
order: 90
---
# Condition to solve a block

For target $\tau\in[0,1]$ we define the following two conditions a header $h$ must satisfy to have a valid Proof of Balanced Work:

1. The Sha256t hash must not be too small: $Y(h) \ge c$ for some constant $c=0.005$
2. The Janushash must be below the target: $J(h) < \tau$.


### Implementation:
The function to check for valid Proof of Balanced Work can be implemented as follows:
```c++
bool valid_pow(Header h, Target t)
{
    auto sha256tFloat { CustomFloat(hashSHA256(hashSHA256(hashSHA256(h)))) };
    auto verusFloat { CustomFloat(verus_hash(h)) };
    constexpr auto c = CustomFloat(-7, 2748779069); // c = 0.005
    if (sha256tFloat < c) {
        return false
    }
    constexpr auto exponent { CustomFloat(0, 3006477107) }; // = 0.7 
    auto hashProduct { verusFloat * pow(sha256tFloat, exponent) };
    return hashProduct < t;
}
```

The `CustomFloat` class has it's own Repository [here](https://github.com/CoinFuMasterShifu/CustomFloat). It is a portable floating point representation with math functions (`log2`, `pow2`, `pow`) and supports:
- Conversion of a hash into `CustomFloat` number representation.
- Multiplication
- Raising a number to some exponent

The use of CustomFloat is necessary since C math library functions cannot be used in consensus-critical parts of a cryptocurrency. This way portatility is guaranteed. 
