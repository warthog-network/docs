---
order: 100
---
# Hashes as Numbers

- Each hash (32 byte hex string) can be interpreted as a number in the interval $[0,1)$. 
    - For example the hash `FFFF...FFFF` would correspond to 1-2^256 which is almost 1 and 
    - the hash `0000...0000` would correspond to 0. 
    - **Note**: A hash cannot correspond to exactly 1 but almost 1. 
- For a block header $h$ we denote
    - the verus hash interpreted as a number in $[0,1)$ by $X(h)$ and
    - the triple sha256 hash interpreted as a number in $[0,1)$ by $Y(h)$.
- We define the Janushash number $J(h) = X(h)Y(h)^{0.7}$.
- Similar to above where we converted a hash to a number we can do the reverse, i.e. interpret $J(h)$ as a hash, this hash is the Janushash but we will never compute it or work with it, instead we will solely consider the number representation $J(h)$. All theory works with numbers, so implementation only needs to convert hashes to numbers, but not numbers to hashes For convenience we will call the Janushash number also *Janushash*.
!!!
To represent a small number as a hash, one might require more than 32 digits, there exist transcendental numbers which even require infinite digits to be represent exactly. However this is not of interest for us.
!!!
