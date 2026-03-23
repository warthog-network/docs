---
order: 85
---
# Mining

Optimally mining janushash exhausts both, a system's CPU and GPU to their limits. GPU is more efficient at Sha256t computations while CPU is more efficient at Verushash v2.2. Since Sha256t hashrate will usually be larger than Verushash v2.2 hashrate by orders of magnitude, it is important to decide optimally on which headers we evaluate the Verushash v2.2 hash function.

Since Sha256t and Verushash v2.2 are different proper hash functions the result of one does not correlate with the result of the other (mathematically we can model the two hash functions' outcomes as *independent*). Therefore to minimize the janushash value  (which needs to be below the target to mine a block)

`verushash(header)*sha256t(header)^0.7`

it is best to evaluate `verushash` on the headers with smallest `sha256t` values that are still larger than the constant c to match the first mining condition above:

1. Sha256t(header) > c for some hard-coded constant c.

In particular out of the many GPU computed sha256t hashes we need to select the smallest that are greater than c. We select just as many as the CPU can handle. 

This gives us the following approach:

## Mining Approach

1. Mine sha256t of headers on GPU
2. Send those headers with sha256t > c and smaller than c + hr_CPU/hr_GPU into a queue that is processed by CPU to evaluate verushash on them.
3. Compute Verushash v2.2 on these headers
4. Evaluate janushash number representation `verushash(header)*sha256t(header)^0.7` and check if it is smaller than the target `t`.

!!!
Above in 2. we use the quotient of CPU and GPU hashrates on verushash, sha256t respectively. The idea behind the band [`c`, `c+hr_CPU/hr_GPU`] is that on average, the number of sha256t hashes that fall into this band will be at rate `hr_CPU`, so this strategy will produce header candidates in the CPU queue at exactly the rate the CPU can handle.
!!!

!!!
To evaluate janushash number representation in 4. above, we should copy not only the headers but also the first 4 bytes of the evaluated sha256t hash from GPU device to host memory. Otherwise we would need to evaluate `sha256t(header)` again.
!!!
