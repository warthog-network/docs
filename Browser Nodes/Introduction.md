---
label: Introduction
order: 100
---

# Browser Nodes
# Overview
Warthog is the first and only crypto project that brought full nodes to the browser. This is possible by harnessing latest technology such as [OPFS](https://sqlite.org/wasm/doc/trunk/persistence.md#opfs) and [SQLite's recent support for it](https://sqlite.org/wasm/doc/trunk/persistence.md#opfs).

There are 3 types of Warthog nodes:

Node type   | Connection Protocols
---    | ---
Normal | TCP
Bridge | TCP, Websocket, (later WebRTC support)
Browser | Websocket, (later WebRTC support)

For now browser nodes cannot communicate between each other and are only connected to bridge nodes. Warthog nodes of version 0.7.47 or above can be used as a bridge node.
[!ref](/Guides/bridge-node-setup.md)


## Frequently asked questions
Question   | Answer
---    | ---
*What are browser nodes?* | Browser nodes are Warthog nodes that should run on every modern browser and run by just opening the website https://node.warthog.network .
*Are browser nodes full nodes?* | Yes. They save and verify the whole chain.
*How much data is used by a browser node?* | Browsers nodes are full nodes and need the same amount of space as normal nodes. This is currently around 1.3 GB of data.
*Where do browser nodes save the chain data?* | Browsers support the [Origin Private File System](https://developer.mozilla.org/en-US/docs/Web/API/File_System_API/Origin_private_file_system) (OPFS) which contains the chain.db3 file.
*How can I access the OPFS file system?* | OPFS is very new and browsers don't offer a method to browse and access it easily. For Chrome browser there exists the [OPFS explorer](https://chromewebstore.google.com/detail/opfs-explorer/acndjpgkpaclldomagafnognkcgjignd) to view files.
*Does the browser node need to be fully synchronized, or can I copy the chain.db3 from another node?* | Not now, maybe in the future browsers offer a way to copy files into your browser's OPFS.
*Do I need to open the P2P port of the firewall to speed up the synchronization? Like port 9186.* | No, synchronization speed depends on the performance of your device and on the number of bridge nodes.
*Do browser nodes communicate peer to peer?* | For now browser nodes only communicate to bridge nodes, in the future we want to add P2P communication over WebRTC.
