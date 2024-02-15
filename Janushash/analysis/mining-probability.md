---
order: 90
---
# Mining Probability
Recall that a block is rejected if the Sha256t hash of its header is too small, i.e. if $Y< c$. Furthermore, the smaller $Y$ the easier it is to satisfy the second condition $XY^{0.7}< \tau$ because larger, and therefore easier-to-mine Verushash v2.1 hashes are accepted.

Now consider a specific mining setting. We denote the Verushash v2.1 hashrate by $\mathfrak{h}_X$ and the Sha256t hashrate by $\mathfrak{h}_Y$. For simplicity we will call $\mathfrak{h}_X$ the *CPU hashrate* and $\mathfrak{h}_Y$ the *GPU hashrate* because these are the devices that the respective algorithms are typically mined on at the moment. We will denote the $\frac{\mathfrak{h}_Y}{\mathfrak{h}_X}$ by $a$ and since GPU hashrate is usually greater than CPU hashrate $a$ will be greater than 1. We call this number the *hashrate ratio*.

To match CPU hashrate, hashes computed on GPU must be filtered, and from the discussion above a reasonable filter condition is to compute Verushash v2.1 on headers $h$ that satisfy

$$ 

c< Y(h) < c+1/a

$$

This way we would select fraction $1/a$ of GPU hashes to check Verushash v2.1 on the corresponding headers. The fraction $1/a$ of GPU hashrate will exactly match the CPU hashrate such that filtered GPU results will come at the right rate to be processed by CPU. 

Note that this is only true for $a > (1-c)^{-1}$, for the small range between $1$ and $(1-c)^{-1}$ (which is only a tiny bit larger than 1) the above reasoning would need to treat the case where filtering cannot avoid that some GPU hashes are rejected for $Y$ being smaller than $c$ if we want to match CPU rate. In this case CPU and GPU hash rates are just too close to allow enough filtering. But we ignore this small range for now as usually GPU hash rate on Sha256t is orders of magnitude larger than CPU hashrate on Verushash v2.1.

For some number $d \in [c,1]$ it holds that

$$
\begin{align*} 
\mathbb{P}\big[(X,Y)\in A_{\tau}\land Y\in[c,d]\big]&=\int_{-\log(d)}^{-\log(c)}e^{-y}\int_{-\log(\tau)-0.7y}^{\infty}e^{-x}\textnormal{d}x\textnormal{d}y\\
&=\int_{-\log(d)}^{-\log(c)}e^{-(1-0.7)y}\tau\textnormal{d}y\\
&=
\tfrac{1}{1-0.7}\big(d^{1-0.7}-c^{1-0.7}\big)\tau\\
&=\tfrac{10}{3}\tau(d^{0.3}-c^{0.3}) \\
\mathbb{P}\big[Y\in[c,d]\big]&=\int_{\log(d)}^{-\log(c)}e^{-y}\textnormal{d}y=(d-c)\\
\mathbb{P}\big[(X,Y)\in A_{\tau}\vert Y\in[c,d]\big]&=\frac{\mathbb{P}\big[(X,Y)\in A_{\tau}\land Y\in[c,d]\big]}{\mathbb{P}\big[Y\in[c,d]\big]}\\
&=\tfrac{10}{3}\tau\frac{(d^{0.3}-c^{0.3})}{(d-c)}
\end{align*}
$$

We denote the conditional probability to mine a block for  $Y(h)$ filtered to be in the interval $[c, c+1/a]$ by $p_{\tau}(a)$. If we plug in $d=c+1/a$ above, we observe that

$$
p_{\tau}(a) = \mathbb{P}\big[(X,Y)\in A_{\tau}\vert Y\in[c,c+1/a]\big] =\tfrac{10}{3} a\big((c+1/a)^{0.3}-c^{0.3}\big)\tau
$$

