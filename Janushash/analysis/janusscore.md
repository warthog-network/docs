---
label: Janusscore
order: 80
---

# Janusscore - a combined hashrate equivalent
We define the *Janusscore* $S(\mathfrak{h}_X,\mathfrak{h}_Y)$ for Verushash v2.1 hashrate $\mathfrak{h}_X$ and Sha256t hashrate $\mathfrak{h}_Y$ by

$$
S(\mathfrak{h}_X,\mathfrak{h}_Y)= \tfrac{10}{3}\big((c+\tfrac{\mathfrak{h}_X}{\mathfrak{h}_Y})^{0.3}-c^{0.3}\big)\mathfrak{h}_Y.
$$

With this definition we can express the expected number mined blocks with traget $\tau$ in time $t$ as
$$
p_{\tau}(\tfrac{\mathfrak{h}_Y}{\mathfrak{h}_X})\mathfrak{h}_Xt = \tfrac{10}{3}\tfrac{\mathfrak{h}_Y}{\mathfrak{h}_X}\big((c+\tfrac{\mathfrak{h}_X}{\mathfrak{h}_Y})^{0.3}-c^{0.3}\big)\tau t = S(\mathfrak{h}_X,\mathfrak{h}_Y)\tau t.
$$
This means two things:
1. For every target $\tau$ the expected number of mined blocks in time $t$ is proportional to $S(\mathfrak{h}_X,\mathfrak{h}_Y)$ and therefore $S(\mathfrak{h}_X,\mathfrak{h}_Y)$ describes the mining efficiency.

2. $S(\mathfrak{h}_X,\mathfrak{h}_Y)$ takes the role of a hashrate. For example one can estimate the Janusscore $S$ by dividing the number of mined blocks by the time and the target (this can be used in pools to estimate the Janusscore based on number of shares, time and difficulty).


The Janusscore the unit "hashes per second" and can be interpreted as a hashrate equivalent to compare different setups.

!!!
Increasing one of the hashrates of $\mathfrak{h}_X$, $\mathfrak{h}_Y$ while leaving the other constant will always increase the Janusscore.
!!!

==- Python code to compute the Janusscore

```python
# define Janusscore function
c = 0.005
S = lambda hx, hy: hy * 10 * ((c + hx/hy)**0.3 - c**0.3)/3

# example usage with 10 mh/s Verushash hashrate and 250 mh/s Sha256t hashrate
S(10000000, 250000000)
```
===

==- Julia code to compute the Janusscore

```julia
# define Janusscore function
c = 0.005
S(hx, hy) = hy * 10 * ((c + hx/hy)^0.3 - c^0.3)/3

# example usage with 10 mh/s Verushash hashrate and 250 mh/s Sha256t hashrate
S(10000000, 250000000)
```
===


[^1]: *CoinFuMasterShifu* (2023). **[Proof of Balanced Work: The Theory of Mining Hash Products](https://github.com/CoinFuMasterShifu/ProofOfBalancedWork/blob/main/PoBW.pdf)**
