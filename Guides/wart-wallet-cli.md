# Use a CLI wallet

![](/img/get-started/10-wallet.png)

## Connect to node (local or public)

First, you need a node :
You can follow this guide to run a local node [here](/docs/blob/master/Guides/node.md)

Or you can connect your wallet to a [public node](https://github.com/warthog-network/public-nodes)
Then, to connect to a public node, you will have to add on your commands the arguments `-h` for the node IP or URL and `-p` for the node port.

For example :
```
./wart-wallet -b -h 193.218.118.57 -p 3001
```

## Create wallet

To create the wallet, just run this command:
```
./wart-wallet -c
```
In output you get wallet address, private and public keys. This information will be saved to the wallet.json file. It is not encrypted and other malicious programs you run on your computer can steal it. Back it up. Don't show it to anyone. The private key can be used to restore the wallet.

To print your address use the command 
```
./wart-wallet -a
```
## Check balance

To check your balance use
```
./wart-wallet -b
```
## Send coins

To send some coins, use command:
```
./wart-walet -s --to=%address% --amount %count% --fee 0
```
Example :
```
./wart-walet -s --to=9da0dc31d0c3bcaaeaf422be6499fbf451b314fa622be978 --amount 0.00000001 --fee 0
```
Next, you will be shown the details of the transaction and if you agree with them, you need to confirm it
```
Summary:
   To:     9da0dc31d0c3bcaaeaf422be6499fbf451b314fa622be978
   Fee:    0.00000001
   Amount: 0.00000001
   Nonce: 4262777042
Confirm with "y": y
```
And your transaction added to blockchain
```
Get pin
Got pin
NonceId: 4262777042
pinHeight: 896450
pinHeight: 896450
pinHash: 2452592e211a10b97da31e4b681aafe49b93321776580278b0c39fc5bae84558
=====DEBUG INFO TRANSACTION JSON=====
{
   "amount": "0.00000001",
   "fee": "0.00000001",
   "nonceId": 4262777042,
   "pinHeight": 896450,
   "signature65": "53d429fa34ef05f9f413b4eb57...9c189ae3078894cb01aee91eda2201",
   "toAddr": "9929fdce19b2a71cf0950263baa71478ef1980f8c7d9ccb3"
}
=====================================
Transaction accepted.
```
## Restore

To restore your wallet.json file, you can run command:
```
./wart-wallet -r %your private key%
```
In output you get wallet address, private and public keys and
```
./wart-wallet -r 2cc2cbe28cb665d540c949aef5d1a3484a960740338780ec82a9ae1414bcc18a
Wallet file created.
{
   "address": "5fe016f874698e11f36a2b2f4dd759d53b55d5520c255e64",
   "privateKey": "2cc2cbe28cb665d540c949aef5d1a3484a960740338780ec82a9ae1414bcc18a",
   "publicKey": "0285a33afacea56262b44b8b96f4affa8f31cc51d3065eb37e43c0d0cf979a2872"
}
```

## Get more help

For more information on how to use the wallet read the `--help` section:
```
./wart-wallet --help
```
![](/img/get-started/11-wallet-help.png)
