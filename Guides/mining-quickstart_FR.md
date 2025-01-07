# Warthog - Tutoriel de minage - Mis à jour le 24/04/2024

Ce guide contient toutes les infos nécessaires pour vous lancer immédiatement dans le minage de Warthog, en pool ou en solo.

## Liens :

- Site : http://www.warthog.network/
- Liens : https://wartscan.io/links
- Pools et exchanges : https://miningpoolstats.stream/warthog
- Calculateur de minage : https://wartscan.io/calculator

## Créer votre wallet :

:warning: Il est déconseillé de miner sur une adresse d'exchange. Vous pouvez potentiellement perdre vos coins minés. Installez votre propre wallet.

### wart-dapp

![wart-dapp](/img/dapp/02-overview.png)

- Wallet GUI (Graphique) : conseillé aux débutants
- Node nécessaire (local ou [publique](https://github.com/warthog-network/public-nodes)), connexion rapide à un node publique possible
- Téléchargement : https://github.com/warthog-network/wart-dapp/releases
- Guide d'utilisation : https://docs.warthog.network/guides/wart-dapp/

### wart-wallet (GUI)

![wart-wallet-GUI](/img/wart-wallet-GUI.png)

- Wallet GUI (Graphique) : conseillé aux débutants
- Node nécessaire (local ou [publique](https://github.com/warthog-network/public-nodes))
- Téléchargement : https://github.com/warthog-network/wart-wallet/releases

### wart-wallet (CLI)

![wart-wllet](/img/get-started/10-wallet.png)

- Wallet CLI (en ligne de commande)
- Node nécessaire (local ou [publique](https://github.com/warthog-network/public-nodes))
- Téléchargement : https://github.com/warthog-network/Warthog/releases
- Guide d'utilisation : https://docs.warthog.network/guides/wart-wallet-cli/

## Hardware pour miner :
L'algo janushash en Proof of Balanced Work favorise les configs type PC gamer (bon CPU impératif + bon GPU) plutôt que les rigs multi-GPU. Le mineur vous donnera le hashrate GPU, le hashrate CPU, et le janus hr qui dépend des deux. C'est ce dernier qui est votre "vrai" hashrate et apparaît sur la pool.

:warning:  **Utilisation de risers déconseillée** : perte de hashrate possible. Faites des tests avec et sans si besoin.

## Mineurs compatibles :

### bzminer (recommandé) :  https://www.bzminer.com/

![bzminer](/img/screen_bzminer.png)

- En développement actif
- Recommandé pour la plupart des cas
- Support du Warthog sur les **versions 21.0.3 et supérieures**
- ℹ️ Versions beta disponibles [ici](https://www.bzminer.com/betas/). Elles peuvent vous donner de meilleurs performances en avance par rapport à la version stable. (faites des essais)

### Janusminer : https://github.com/CoinFuMasterShifu/janusminer

![janusminer](/img/screen_janusminer.png)

- Mineur historique du projet
- open-source
- développement arrêté depuis le support de bzminer
- Instable sous Windows
- Pas optimisé pour GPU Nvidia


## Minage sous Hiveos :

Version Beta conseillée. Passage à Hiveos Beta : ` hive-replace -y --beta`

## Overclocking GPU

L'algo de minage GPU utilisé est le SHA256t. Cet algo est core intensif : mêmes OC que Radiant, Kaspa, Alph...

Pour connaître le hashrate et l'overclocking de votre GPU sur https://hashrate.no/ , référez-vous au données de votre GPU sur le coin NOVO.
Exemple pour une RTX 3070 : https://hashrate.no/gpus/3070/NOVO

Sous BZminer, possibilité d'utiliser les extra arguments d'overclocking.
Vous pouvez pour cela utiliser le générateur de config : https://www.bzminer.com/config-generator/

## Minage en solo (pour ceux qui le souhaitent. Pools dispo sinon) :

La récompense de bloc est actuellement de **3 WART**
Si vous connaissez votre hashrate Janusscore, vous pouvez utiliser le calculateur pour estimer le nombre de blocs que vous pouvez espérer miner en solo : https://wartscan.io/calculator

### Télécharger le node :
Ici : https://github.com/warthog-network/Warthog/releases

### Lancer le node sous Linux :
  - Le rendre exécutable : `chmod +x wart-node-linux`
  - L'exécuter dans un screen :`screen ./wart-node-linux --stratum=0.0.0.0:3456` (stratum nécessaire pour minage solo avec bzminer)

 ### Lancer le node sous Windows :
Créez un fichier .bat dans le dossier du .exe contenant : `wart-node-windows.exe --stratum=0.0.0.0:3456`  et lancez-le.

### Lancer votre mineur en solo sur votre node : 
Pool à renseigner sur votre mineur solo : `stratum+tcp://ip-du_node:3456`

### Exemple de flight sheet Hiveos :

(Sur node solo, avec overclocking)

![bzminer-hiveos](/img/bzminer_hiveos.png)

### Mining profil mmpOS :

![bzminer-mmpOS](/img/bzminer_mmpos.png)
