# Warthog - Update 05/04/2024 @here

## Liens :

- Site : http://www.warthog.network/
- Liens : https://wartscan.io/links
- Pools/exchanges : https://miningpoolstats.stream/warthog

## Wallets :
- wart-dapp (GUI, prêt à l'emploi, connexion auto à node public possible) : https://github.com/warthog-network/wart-dapp/releases
- en ligne de commande (avec node) : https://github.com/warthog-network/Warthog/releases

## Miner :
L'algo janushash en Proof of Balanced Work favorise les configs type PC gamer (bon CPU impératif + bon GPU) plutôt que les rigs multi-GPU. Le mineur vous donnera le hashrate GPU, le hashrate CPU, et le janusscore qui dépend des deux. C'est ce dernier qui est votre "vrai" hashrate et apparaît sur la pool.

:warning:  **Utilisation de risers déconseillée**

### BZminer (recommandé) :  https://www.bzminer.com/

## Hiveos :

- Passage à Hiveos Beta : ` hive-replace -y --beta`
- MAJ de BZminer :
```
wget https://bzminer.com/downloads/bzminer_v21.0.3_linux.tar.gz; tar -xvf bzminer_v21.0.3_linux.tar.gz; miner stop; cp bzminer_v21.0.3_linux/bzminer /hive/miners/bzminer/20.0.0/; miner start
```

## OC GPU

Core intensif : mêmes OC que Radiant, Kaspa, Alph...
Sous BZminer, possibilité d'utiliser les extra arguments.

## Minage en solo (pour ceux qui le souhaitent. Pools dispo sinon) :
Télécharger le node : https://github.com/warthog-network/Warthog/releases

### Lancer le node sous Linux :
  - Le rendre exécutable : `chmod +x wart-node-linux`
  - sous Linux :`wart-node-linux --stratum=0.0.0.0:3456` (stratum nécessaire pour minage solo avec bzminer)

 ### Lancer le node sous Windows :
créez un .bat dans le dossier du .exe contenant : `wart-node-windows.exe --stratum=0.0.0.0:3456`  et lancez-le :slight_smile:
- Pool à renseigner sur votre mineur solo : `stratum+tcp://ip-du_node:3456`

### FS hiveos :
Le force algo ne sera plus nécessaire une fois warthog proposé directement comme algo.
URL de la pool de votre choix à renseigner, ou IP de votre node.
