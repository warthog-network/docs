!!!
Browser nodes feature is techincally almost completely implemented but has lower priority for completion than the DeFi progress. This feature is not complete yet.
!!!

# Browser Nodes

## Overview

Warthog is the first and only crypto project that brought full nodes to the browser. This is achieved by compiling the native C++ node code to WebAssembly and using modern browser storage APIs for persistence. Simply open a website to run a complete Warthog node in your browser.

## Technical Architecture

### WebAssembly

The Warthog node is compiled to WebAssembly (WASM), a binary instruction format that runs in all modern browsers. This allows the same C++ codebase that powers normal and bridge nodes to run entirely client-side in the browser without plugins or extensions.

### Origin Private File System (OPFS)

Browser nodes persist the blockchain data using the Origin Private File System (OPFS), part of the File System API. OPFS provides:
- **Private storage**: Data is scoped to the originating website and not accessible to other sites
- **Persistent files**: Files survive browser restarts and page reloads
- **Synchronous access**: Fast read/write operations suitable for database access

The entire blockchain database (`chain.db3`) is stored in OPFS as a SQLite database file. This means browser nodes maintain a complete, verified copy of the blockchain without relying on external servers.

### WasmFS

WasmFS extends WebAssembly's virtual filesystem to leverage OPFS under the hood. This provides transparent file persistence for the WASM module, enabling the node to maintain synchronization state across sessions.

## Node Types

Warthog supports three node types, each with different capabilities:

| Node Type | Protocols | Storage | Use Case |
| --- | --- | --- | --- |
| Normal | TCP | Local filesystem | Standard full node operation |
| Bridge | TCP, WebSocket | Local filesystem | Gateway between TCP and WebSocket/WebRTC clients |
| Browser | WebSocket | OPFS (browser) | Lightweight client-side full node |

Bridge nodes act as intermediaries, translating between different network protocols. They can be run by anyone running Warthog version 0.7.47 or later.

## P2P Communication

### Current State
Browser nodes currently connect to bridge nodes via WebSocket. This allows them to participate in the network and synchronize the blockchain, but direct browser-to-browser communication is not yet active.

### WebRTC Support (Future)
Planned WebRTC support will enable direct peer-to-peer communication between browser nodes without requiring bridge nodes as intermediaries. This involves:

- **Signaling**: A protocol for browsers to discover and establish connections with each other
- **NAT traversal**: Techniques to punch through firewalls and router configurations for direct connections
- **Connection negotiation**: Handshaking to agree on connection parameters

The advantage of WebRTC is that it allows browsers to communicate directly, similar to how desktop nodes communicate over TCP, reducing dependence on bridge nodes and improving network decentralization.

## Synchronization

Browser nodes synchronize with the network through bridge nodes. The process:

1. Connect to one or more bridge nodes via WebSocket
2. Request chain headers and blocks
3. Verify all blocks and transactions locally
4. Store verified data in OPFS

Synchronization speed depends on device performance and the number of available bridge node connections. Unlike normal nodes, browser nodes do not need to open any firewall ports.

## Storage Requirements

Browser nodes are full nodes and require approximately 1.3 GB of storage in OPFS. This stores the complete blockchain history, not just headers or recent transactions.

## Frequently Asked Questions

### What are browser nodes?
Browser nodes are Warthog nodes that run directly in any modern browser by visiting https://node.warthog.network. No installation or configuration required.

### Are browser nodes full nodes?
Yes. Browser nodes download and verify the entire blockchain, just like normal nodes. They independently validate all transactions and blocks.

### How much data does a browser node use?
Browser nodes are full nodes and require approximately 1.3 GB of storage, the same as normal nodes.

### Where does the browser node save chain data?
Browser nodes use the Origin Private File System (OPFS) to store the chain database file (chain.db3). This storage is private to the website and persists across sessions.

### How can I access the OPFS filesystem?
OPFS is not easily accessible through standard browser interfaces. For Chrome, you can use the [OPFS Explorer](https://chromewebstore.google.com/detail/opfs-explorer/acndjpgkpaclldomagafnognkcgjignd) extension to view files.

### Can I copy chain data from another node?
Currently, browsers do not provide a way to import files into OPFS programmatically. Future versions may allow importing a chain snapshot to speed up initial synchronization.

### Do I need to open firewall ports for browser nodes?
No. Browser nodes connect outbound to bridge nodes via WebSocket, so no inbound ports need to be opened.

### Will browser nodes support direct P2P communication?
Yes, WebRTC support is planned for future versions, enabling direct browser-to-browser communication without bridge nodes.
