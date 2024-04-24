# Warthog - Mining quick start guide - Updated 2024-04-22

This guide contains all needed information to start mining Warthog, on your own node or on pool.

## Links :

- Website : http://www.warthog.network/
- Links : https://wartscan.io/links
- Pools and exchanges : https://miningpoolstats.stream/warthog
- Mining calculator : https://wartscan.io/calculator

## Create your wallet :

:warning: Mining on an exchange address is risky. You can potentially lose your mined coins. Install your own wallet.

### wart-dapp

![wart-dapp](/img/dapp/02-overview.png)

- GUI (Graphical) : Easier for beginners
- Can be connected to a public node (No need to install a local node)
- Download : https://github.com/warthog-network/wart-dapp/releases
- Guide : https://www.warthog.network/docs/wart-dapp/

### wart-wallet

![wart-wllet](/img/get-started/10-wallet.png)

- CLI (command-line)
- Local node needed
- Download : https://github.com/warthog-network/Warthog/releases
- Guide : https://www.warthog.network/docs/

## Mining hardware :
The janushash Proof of Balanced Work algo works better with gaming PC hardware (Good CPU needed + good GPU) than with multi-GPU rigs. The miner will give you the GPU hashrate, the CPU hashrate, and the janus hashrate with depend of the two of them. The janus hashrate is your "real" hashrate and the one you will see on your pool.

:warning:  **Avoid to use GPU risers** : They can cause low GPU hashrate. Do tests with and without if needed.

## Compatible miners :

### bzminer (recommanded) :  https://www.bzminer.com/

![bzminer](/img/screen_bzminer.png)

- Active development
- Recommended for most cases
- Warthog is supported on **versions 21.0.3 and higher**

### Janusminer : https://github.com/CoinFuMasterShifu/janusminer

![janusminer](/img/screen_janusminer.png)

- First miner of the project
- open-source
- developement stopped since bzminer support
- Instable on Windows
- Not optimized for Nvidia GPUs


## Mining on Hiveos :

Beta version is adviced. Command to upgrade : ` hive-replace -y --beta`

## GPU Overclocking

L'algo de minage GPU utilisé est le SHA256t. Cet algo est core intensif : mêmes OC que Radiant, Kaspa, Alph...

Pour connaitre le hashrate et l'overclocking de votre GPU sur https://hashrate.no/ , référez-vous au données de votre GPU sur le coin NOVO.
Exemple pour une RTX 3070 : https://hashrate.no/gpus/3070/NOVO

Sous BZminer, possibilité d'utiliser les extra arguments d'overclocking.
Vous pouvezpour cela utiliser le générateur de config : https://www.bzminer.com/config-generator/

## Minage en solo (pour ceux qui le souhaitent. Pools dispo sinon) :

La récompense de bloc est actuellement de **3 WART**
Si vous connaissez votre hashrate Janusscore, vous pouvez utiliser le calculateur pour estimer le nombre de blocs que vous pouvez espérer miner en solo : https://wartscan.io/calculator

### Télécharger le node :
Ici : https://github.com/warthog-network/Warthog/releases

### Lancer le node sous Linux :
  - Le rendre exécutable : `chmod +x wart-node-linux`
  - L'exécuter dans un screen :`screen ./wart-node-linux --stratum=0.0.0.0:3456` (stratum nécessaire pour minage solo avec bzminer)

 ### Lancer le node sous Windows :
créez un fichier .bat dans le dossier du .exe contenant : `wart-node-windows.exe --stratum=0.0.0.0:3456`  et lancez-le.

### Lancer votre mineur en solo sur votre node : 
Pool à renseigner sur votre mineur solo : `stratum+tcp://ip-du_node:3456`

### Exemple de flight sheet Hiveos :

(Sur node solo, avec overclocking)

![bzminer-hiveos](/img/bzminer_hiveos.png)

### Mining profil mmpOS :

![bzminer-mmpOS](/img/bzminer_mmpos.png)
