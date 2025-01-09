# Warthog - Mining quick start guide - Updated 2024-04-24

This guide contains all needed information to start mining Warthog, on your own node (solo) or on pool.

## Links :

- Website : http://www.warthog.network/
- Links : https://wartscan.io/links
- Pools and exchanges : https://miningpoolstats.stream/warthog
- Mining calculator : https://wartscan.io/calculator

## Create your wallet :

:warning: Mining on an exchange address is risky. You can potentially lose your mined coins. Install your own wallet.

### wart-dapp (GUI)

![wart-dapp](/img/dapp/02-overview.png)

- GUI (Graphical) : Easier for beginners
- Node needed (local or [public](https://github.com/warthog-network/public-nodes)), quick connection to public node available
- Download : https://github.com/warthog-network/wart-dapp/releases
- Guide : https://docs.warthog.network/guides/wart-dapp/

### wart-wallet (GUI)

![wart-wallet-GUI](/img/wart-wallet-GUI.png)

- GUI (Graphical) : Easier for beginners
- Node needed (local or [public](https://github.com/warthog-network/public-nodes))
- Download : https://github.com/warthog-network/wart-wallet/releases

### wart-wallet (CLI)

![wart-wallet](/img/get-started/10-wallet.png)

- CLI (command-line)
- Node needed (local or [public](https://github.com/warthog-network/public-nodes))
- Download : https://github.com/warthog-network/Warthog/releases
- Guide : https://docs.warthog.network/guides/wart-wallet-cli/

## Mining hardware :
The janushash Proof of Balanced Work algo works better with gaming PC hardware (Good CPU needed + good GPU) than with multi-GPU rigs. The miner will give you the GPU hashrate, the CPU hashrate, and the janus hashrate with depend of the two of them. The janus hashrate is your "real" hashrate and the one you will see on your pool.

:warning:  **Avoid to use GPU risers** : They can cause low GPU hashrate. Do tests with and without if needed.

## Compatible miners :

### bzminer (recommanded) :  https://www.bzminer.com/

![bzminer](/img/screen_bzminer.png)

- Active development
- Recommended for most cases
- Warthog is supported on **versions 21.0.3 and higher**
- ℹ️ Find beta versions [here](https://www.bzminer.com/betas/). They can give you early better performances compared to stable release. (do some testing)

### Janusminer : https://github.com/CoinFuMasterShifu/janusminer

![janusminer](/img/screen_janusminer.png)

- First miner of the project
- Open-source
- Development stopped since bzminer support
- Unstable on Windows
- Not optimized for Nvidia GPUs


## Mining on Hiveos :

Beta version is adviced. Command to upgrade : ` hive-replace -y --beta`

## GPU Overclocking

The GPU mining algo is SHA256t. It is a core intensive algo : use same OC than Radiant, Kaspa, Alph...

To know your GPU hashrate and overclocking on https://hashrate.no/ ,you can use your GPU data on the NOVO coin.
Example with RTX 3070 : https://hashrate.no/gpus/3070/NOVO

On BZminer, you can use the overclocking extra arguments.
For that, you can use the config generator : https://www.bzminer.com/config-generator/

## Solo mining (For those who want it. Pools are available else) :

The current block reward **3 WART**
If you know your Janusscore hashrate, you can use the calculator to estimate the number of blocks you can hope to mine on solo : https://wartscan.io/calculator

### Download the node :
Here : https://github.com/warthog-network/Warthog/releases

### Run the node on Linux :
  - Make it executable : `chmod +x wart-node-linux`
  - Run it on a screen :`screen ./wart-node-linux --stratum=0.0.0.0:3456` (stratum argument needed to mine solo with bzminer)

 ### Run the node on Windows :
Create a .bat file on the .exe folder, containing : `wart-node-windows.exe --stratum=0.0.0.0:3456`  and run it.

### Run your miner on your node (solo) : 
Pool syntax to write on your solo miner : `stratum+tcp://your-node-local-ip:3456`

### Hiveos example of flight sheet :

(On a solo node, with overclocking)

![bzminer-hiveos](/img/bzminer_hiveos.png)

### mmpOS example of mining profile :

![bzminer-mmpOS](/img/bzminer_mmpos.png)
