# Create a CLI wallet

First, you need a node :
You can follow this guide to run a local node [here](/docs/blob/master/Guides/node.md)

Or you can connect your wallet to a [public node](https://github.com/warthog-network/public-nodes)
Then, to connect to a public node, you will have to add on your commands the arguments `-h` for the node IP or URL and `-p` for the node port.

For example :
```
./wart-wallet -b -h 193.218.118.57 -p 3001
```

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

![](/img/get-started/10-wallet.png)

For more information on how to use the wallet read the `--help` section:
```
./wart-wallet --help
```
![](/img/get-started/11-wallet-help.png)
