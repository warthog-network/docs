# Welcome

Warthog is an experimental L1-cryptocurrency implementation without specific focus. This project is not a dumb fork of something else. It was developed completely from scratch!

Warthog's *Janushash* mining algorithm is unique across the crypto space, it is the world's first [Proof of Balanced Work (PoBW) algorithm](https://github.com/CoinFuMasterShifu/ProofOfBalancedWork/blob/main/PoBW.pdf), which requires CPU and GPU hash power for mining.

The [core repository](https://github.com/warthog-network) offers two applications:
* Node
* Command line Wallet

Furthermore there are two third-party wallets:
* [Wart-dapp](https://github.com/warthog-network/wart-dapp) (open source)
* [wart-wallet](https://github.com/warthog-network/wart-wallet) (open source)

You can join the Warthog community on [Discord](https://discord.com/invite/QMDV8bGTdQ).



## Starting a node

Doc available here : https://github.com/Tintin-BZH/docs/blob/master/Guides/node.md

## Using the CLI Wallet
We explain how to use the command line wallet. If you don't know how to use the command line, you should use a graphical wallet such as [Wart-dapp](https://github.com/warthog-network/wart-dapp) or [wart-wallet](https://github.com/warthog-network/wart-wallet).

Let's create a new wallet
```
./wart-wallet -c
```
A new wallet called `wallet.json` file is created. It is not encrypted and other malicious programs you run on your computer can steal it. Back it up. Don't show it to anyone. The private key can be used to restore the wallet. 
To print your address use the command 
```
./wart-wallet -a
```

To check your balance use
```
./wart-wallet -b
```

You can send some coins with
```
./wart-wallet -s
```
The wallet asks for the address you want to send to, the fee and the amount. You can type in values like these:
Then confirm with "y" and press enter. Of course you cannot send because your wallet address has no coins yet.

![](img/get-started/10-wallet.png)

For more information on how to use the wallet read the `--help` section:
```
./wart-wallet --help
```
![](img/get-started/11-wallet-help.png)

## Mining
Mining Warthog's Janushash algorithm requires both, a good CPU and a GPU. Solo and pool miners can be found here:
* [Janusminer](https://github.com/CoinFuMasterShifu/janusminer) (open source)

We have a guide on how to install OpenCL:
[!ref](Guides/installing-opencl.md)
