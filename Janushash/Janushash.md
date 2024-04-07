# Janushash

## Proof of Balanced Work

### Introduction

Compared to classical Proof of Work, the class of *Proof of Balanced Work (PoBW)* algorithms is quite a different beast. Instead of only employing one hash function they combine *multiple* hash functions in a multiplicative way. The mathematical theory of considering a multiplicative combination of hashes, i.e. *hash products*, was established in this [paper](https://github.com/CoinFuMasterShifu/ProofOfBalancedWork/blob/main/PoBW.pdf) for Warthog and currently there is no other crypto project using Proof of Balanced Work for consensus.

Combining different hash functions multiplicatively has the advantage that
1. different hash functions can be mined in parallel devices at *different hashrates* using a multi-stage filtering approach and
2. efficient mining requires mining of *all* involved algorithms for each block in contrast to previous failed attempts to construct multi-algorithm block chains by using individual difficulties for each algorithm (Myriad Coin, DigiByte, Verge). For example Verge could be hacked by focusing only on one algorithm.

### Janushash
Essentially, Proof of Balanced Work algorithms are simply the multiplicative combinations of existing hash functions. Warthog Janushash algorithm combines two hash functions:

1. triple Sha256 (Sha256t) and
2. Verushash v2.1

### Balancing
Energy-efficient mining of Proof of Balanced Work algorithms requires finding a good balance between Sha256t and Verushash hashrates. The best combination depends on hardware and energy cost but it is clear that mining without a GPU or with a weak CPU won't be competitive. The balancing requirement coined the name "Proof of Balanced Work".

## Hashrate Decentralization 

### Fighting Farms
Interestingly, the Janushash algorithm keeps away both, GPU farms and CPU farms:

GPU farms usually save on the CPU side, because CPU performance is not relevant for mining GPU algorithms. Therefore such farms perform poorly on Janushash. GPU farm owners would need to make significant investments in efficient and performant motherboards and CPUs to improve their GPU/CPU balance for efficiently mining Janushash.

CPU farms perform very poorly on Janushash because of the lack of accelerated triple Sha256 hash evaluation. The same applies to most botnets.

This means large mining farms and botnets play a much smaller role in Warthog than they do in other proof of work cryptocurrencies, which increases decentralization of hashrate. 

### Satoshi's vision

Originally, Satoshi Nakamoto had an idealized hope for mining being a democratized way of establishing consensus. This can be seen for example in his famous [whitepaper](https://bitcoin.org/bitcoin.pdf):
![](../img/one-cpu-one-vote.png)

From [this article](https://insidebitcoins.com/news/laszlo-hanyecs-claims-satoshi-invented-gpu-mining-to-prevent-51-attacks) about Laszlo Hanyecz's correspondence with Satoshi we can observe that Satoshi was not amazed about the fact that GPU mining would disrupt this idealized hope:
- "One of the first emails Satoshi had sent the man was in response to him describing his proposed GPU miner. Mainly, Satoshi was none-too-pleased, asking Hanyecz to slow down with this."
- "Satoshi explained that, at the time, one of the biggest attractions possible is the fact that anyone can download Bitcoin and start mining with their laptops. Without that, it wouldn't have gained as much traction."

He knew that with the advent of GPU mining, many CPU miners would be kicked out of the network, which would be against his vision of fair, equal and decentralized mining. Therefore he hoped to delay this as long as possible.
![](../img/satoshi_bitcointalk.png)

We all know that his hopes have not been fulfilled, today Bitcoin is mined on specialized expensive hardware and only those with access to this hardware can participate in mining. After all, Satoshi was not able to solve the issue of centralized mining.

We are confident that the use of Proof of Balanced Work solves this issue to a large extent when the combined hash functions are carefully selected. In Janushash, the two hash functions Sha256t and Verushash where chosen to require a GPU and a CPU connected with sufficiently large bandwidth. This was done to target typical gaming PCs. As described above, with this choice farms cannot easily join the network without being forced to make additional investments just for mining Warthog. This democratizes mining Warthog and brings it closer to Satoshi's vision. 



## ASIC Resistance
As technology advances, so does specialized mining hardware, especially when potential profits are high. There is nothing that can be done against this fact. However there are three reasons why Warthog is more robust against ASIC threats than other PoW cryptocurrencies:

### Inherited ASIC Resistance

When it comes to ASIC resistance, Proof of Balanced Work is stronger than its strongest ingredient. To accelerate mining, an ASIC would need to be able accelerate computation of all combined hash functions to avoid a bottleneck effect. In addition an ASIC would need enough bandwidth between the hardware sections computing different hash functions as well as calibration and tuning to optimize their intercommunication and coordination.

In particular, Janushash inherits ASIC-resistance from Verushash v2.1 which is currently mined on CPUs and GPUs, but not on FPGAs/ASICs, and the need to also require SHA256t hashrate makes Janushash even more ASIC-resistant.

### Detection of suspicious hashrate

In traditional Proof of Work networks we only have one marker to analyze network hashrate, namely the *network difficulty*. It can be used to estimate the total hashrate of all miners in the network. However we cannot tell whether some actors use specialized hardware to gain an unfair advantage over normal miners.

Janushash however combines two hash functions and harnessing the probability theory and statistics, we can extract information about the Sha256t/Verushash hashrate ratio used to mine a block. This information is shown publicly in the blockchain explorer:

![](../img/hashrateratio.png)

In addition to the network difficulty, this second marker provides useful information on the network hashrate: It allows to spot suspicious hashrate immediately. In Warthog it is much more difficult for ASICs to stay undetected because they must not only successfully mine blocks, but also mimic the hashrate ratio used by honest miners. This is another unique property of Proof of Balanced Work.

### Simple Algorithm Adaption 
The fundamental reason for the favorable properties of the Janushash algorithm is not the particular choice of the combined hash functions itself, but the choice to rely on Proof of Balanced Work to combine different hash functions multiplicatively.
This means that if ASICs really join the network one day, we can simply exchange the combined hash functions, for example for Blake3 on GPU and RandomX on CPU, while preserving all the advantages listed here. Combining established hash functions allows the creation new algorithms fast while benefiting from their maturity and proven properties at the same time. This allows Warthog to adapt quickly when needed.


## Make mining great again
Warthog tries to revive the good old days when mining was fun. The unique properties of the Janushash algorithm help to achieve this goal:

### Escaping one-dimensional mining boredom
In a way, traditional mining in cryptocurrency is one-dimensional, the goal is simply to find the best hardware for evaluating some hash function. In contrast, mining Warthog is two-dimensional: there are two hash functions Sha256t and Verushash v2.1, and both hashrates are relevant for the mining efficiency. This leads to much more versatile options and motivates miners to experiment with endless hardware setups. Vivid discussions about the best combinations bring Warthog mining to life.


### Robin Hood: Steal from the farms to feed the gamers
As explained above, established farms require substantial investments in order to mine Warthog efficiently and making such investment only for mining Warthog might not be reasonable for most farms. 

On the other hand gamers usually have systems with modern platforms and CPUs paired with sufficiently good GPUs to mine Warthog efficiently. Since farms and botnets are less of a direct competitor in Warthog than they are in other Proof of Work cryptocurrencies, this will reflect in increased mining returns for the average gamer or miner, which will in turn contribute to Warthog's popularity.

# TLDR Summary

- Proof of Balanced Work (PoBW) was invented for Warthog and might have a tremendous impact on the whole crypto space.
- Janushash is a PoBW algorithm combining Sha256t and Verushash v2.1
- Efficiently mining Janushash requires both, a GPU and a CPU.
- Miners have to find a good balance of GPU and CPU hardware. Too many GPUs paired with one CPU won't perform well.
- GPU farms cannot easily compete because they usually have weak CPUs. They would need to adapt their motherboards and CPUs just for mining Warthog.
- CPU farms and botnets cannot compete because they can't compute Sha256t reasonably fast.
- Warthog will be interesting for Gamers because Farms won't compete so fiercely.
- Hashrate balance of the two algorithms can be observed from mined blocks, such that additional suspicious hashrate of specialized mining hardware can be easily spotted in the network.
- The above observations imply greater decentralization.
- Due to these properties, Warthog's mining algorithm comes closer to Satoshi's original vision of democratized mining being than any other cryptocurrency.
