# Using a node

For some actions on the Warthog network (using a wallet or solo mining), you need to run an instance of the node on your computer. Currently, the pre-compiled node executable file are availabe for Windows, macOS (only arm), Linux (Ubuntu, Debian). Аfter launch, node will start synchronizing the blockchain, after it is completed, it will be possible to perform operations with the wallet and start solo mining.

## Public nodes

If for some reasons, you cannot use a local node, you can use existing public nodes. They can be found [here](https://github.com/warthog-network/public-nodes)
You can use them to connect your wallet without installing a local node, but **don't use public nodes for solo mining**

## Starting the node

Download the latest version from [here](https://github.com/warthog-network/Warthog/releases).

The node does not have external dependencies, it can be started via command line (file name deending on your platform, Linux, Windows or aarch64) :
```
./wart-node-linux
```
![](/img/get-started/09-node.png)

The node starts syncing. The node needs to be running and synced while we are using the wallet and miner.

## Starting the node with stratum for solo mining

The current miner in development is BZminer. To enable solo mining on your local node, you need to enable node stratum with this argument : 

```
./wart-node-linux --stratum=0.0.0.0:3456
```
(Use the same port on your node stratum and on bzminer, 3456 here for example)
