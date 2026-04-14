# Janushash made simple
The theory behind Warthog's Proof of Balanced Work and specifically the Janushash algorithm is dry and difficult to grasp. In this post we want to give a high-level understanding on how things work.

## Proof of Work

Traditionally Proof of Work (PoW) as implemented in Bitcoin and successors allows cryptocurrency protocols to define the "correct" chain. Since real effort needs to be put into mining blocks secured by PoW, attackers cannot create fake alternative chains.

On a high level PoW mining works by trial and error by inserting different inputs into a _hash function_ (also called "mining algorithm") and hoping for a good output. For every input such a hash function produces an output hash which can be regarded as a number between 0 and 1, and a _good_ output that mines a block can be defined as a very small output number, i.e. very close to 0.
The difficulty can then be seen as a maximal threshold value for hashed number for a valid PoW. 

For example, when the difficulty is 0.1, then blocks with an output hash number smaller than 0.1 have valid PoW and can be appended to the chain. Mining would be the process of changing block content and inserting it into the hash function until we end up with a value smaller than 0.1. 

Normally the outputs are uniformly random, i.e. you cannot predict what input will produce which output and all outputs are equally likely. As a consequence, in this example around 10% of all possible blocks have valid PoW, in other words, an expected number of 10 blocks need to be hashed by a miner until he finds a block.

More generally, the difficulty controls the fraction of blocks that have valid PoW and the expected number of tries a miner has to do before it mines a block.

## Proof of Balanced Work
### Basic Idea

Instead of using one hash function to generate a number between 0 and 1 we can take many hash functions and multiply their outputs. This gives again a number between 0 and 1 because a product of many numbers between 0 and 1 is again between 0 and 1.

Compared to classical mining, this number will not be uniformly random because values close to 1 are less likely to occur in such a product. Luckily uniform randomness is not required for mining to work. Only for retargeting the difficulty it can make a difference but we show in [our paper](https://github.com/CoinFuMasterShifu/ProofOfBalancedWork/blob/main/PoBW.pdf) that in this setting absolutely nothing has to change and the every cryptocurrency would just work normally when their mining algorithm changed from a single hash function to the product of several hash functions.

### Mining PoBW
Mining such a product is a bit different from classical mining: Assuming that the involved hash functions are computed in parallel at fixed possibly different hash rates, it makes sense to use a _filtering approach_ during mining, i.e. to start with the fastest hash function and only compute the second fastest hash function on the candidates with most promising fastest hash function result, then compute the third fastest hash function only on the candidates with most promising second fastest hash result etc. It can be shown that this approach is optimal.

The above strategy is for fixed hashrates. When they are not fixed, but instead we can also choose the proportion of hardware allocated for the evaluation of each hash function, optimizing these proportions such that the combined hashrate gets maximal can be done numerically. As we have shown in our paper, it turns out that for best results the hashrates must be balanced, hence the name _Proof of Balanced Work_ (PoBW). 

### Decentralization through breaking Specialization
At this point you may ask why we should make mining so complicated with different hash functions multiplied, multi-stage mining based on a filtering approach, and optimization problems. The answer to this question is decentralization through breaking specialization.

Over the years, as mining got increasingly professionalized, the ordinary miners lost a lot of ground to the industrial mining farms. We believe that with Proof of Balanced Work, we have found a way to reverse this unfortunate trend and bring crypto mining back to the people. The essential way to achieve this goal is to carefully select the hash functions that we combine in PoBW in a way that cover's consumer hardware's strengths. Since efficient mining requires efficient evaluation of all involved functions, this essentially means that we are defining a _set of strengths_ that a device must support for mining PoBW in contrast to _a single strength_ in traditional PoW. If only one of these strengths is not present, mining will be negatively impacted.

This is the huge advantage that PoBW has to offer: it allows us to target specific hardware and exclude farms from mining efficiently when they don't support all requirements that we picked the combined hash functions for. In this sense PoBW indeed breaks specialized actors' access to mining for the greater good of fair and equal mining democratization bringing us closer to Satoshi's original goal of "one-CPU-one-vote" than ever before.

## Janushash

### Algorithm Design
Janushash is a PoBW algorithm and as such it combines several hash functions. Specifically, to create a mining algorithm which requires both, CPU and GPU, for efficient mining we have carefully selected and combined two hash functions Sha256t and Verushash v2.2. This yields Janushash, Warthog's unique and world's first PoBW mining algorithm, the first one to require CPU and GPU hardware. 

Compared to the general PoBW description above there exists subtle differences in the way the two hash functions are combined  that we implemented for two reasons:

1. **Pool mining**
Fair pool mining requires a method to scale the difficulty such that every participant is affected proportionally irrespective of the individual hashrate combinations they are mining at. Unfortunately, with standard PoBW this is not the case when only the difficulty is scaled by the pool and instead a complicated acceptance region for shares would need to be implemented on the pool side for each difficulty. For the creation of pools and for adaption this is a major barrier which is why a slight modification was necessary.

2. **Diminishing returns:**
For fixed CPU hashrate on Verushash the combined Janushash hashrate is hard-capped no matter how much GPU hashrate we have. This protects the Janushash algorithm from offloading the Sha256t computations to a specialized device like ASICs or FPGAs as their use will provide only marginal improvements over GPU usage when the Verus hashrate is not scaled as well.

### Mining Janushash
Due to hardware strengths it makes sense to evaluate Sha256t hashes on GPUs, and Verushash on CPUs. Since Verus hashrate is usually much lower than Sha256t hashrate, we have to decide depending on Sha256t outputs for which blocks it is worth to also compute the Verus hash because only when we have computed both hashes we can decide whether we mined the block and Verushash is the bottle-neck here. As we want to bring down the hash product, obviously it is optimal to pick those blocks for CPU hashing that have a small Sha256t value and discard those with a high value.

This shows that we need to use the GPU hardware as a filter that decides whether to send a candidate to the CPU side depending on the Sha256t output. The filter needs to be tuned such that it releases candidates exactly at the hashrate that the CPU can process. This implicitly requires a certain bandwidth between GPU and CPU which specifies an additional requirement on the hardware and ties Janushash more to its intended mining environment, i.e. the Gaming PCs.

### The choice of the combined hash functions
We have chosen Verushash particularly over other CPU-friendly hash functions like RandomX because the relatively high hashrate of Verushash implies a reasonably high bandwidth requirement. Without this requirement it would be feasible to create pools that provide non-intended work-sharing, where some participants perform only GPU mining and others do CPU mining. This would destroy Janushash's focus on Gaming PCs, which is the main selling point that excludes specialized GPU farms or botnets and brings us closer to Satoshi's original vision of mining democratization and decentralization.

Furthermore the GPU hash function needs to be clearly faster to evaluate across all scenarios to ensure that it makes sense to use the GPU side as a filter for the CPU evaluation. Since the CPU hashrate on Verushash is in the MH/s range, the GPU hashrate should be in the GH/s range and there are very few such GPU-friendly hash functions. That's why we have chosen Sha256t.
