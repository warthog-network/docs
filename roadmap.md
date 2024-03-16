# Roadmap
## Q2 - Full nodes in web browsers
Warthog is an experimental cryptocurrency where we try to push the boundaries of what is possible in crypto. After having successfully implemented the world's first Proof of Balanced Work mining algorithm we try to port full nodes to the web browser. This has several advantages:
 - Full nodes can be easily spawned on a smartphone just by loading a website.
 - This lowers the entry-barrier for new users.
 - The stability of the network benefits from more nodes.
 - In connection with the light-weight DeFi, ease-of-use on smartphones will be of particular interest.

As of now, to the best of our knowledge there is no other cryptocurrency supporting in-browser full nodes. This may be due to the fact that the technological foundations needed to persist a blockchain in the web browser were only recently developed.

### Persisting the Blockchain

The fact that Warthog relies on SQLite to persist state makes it possible to store the database in the Origin Private File System using WasmFS. The technology is very new:

- [Emscripten's WasmFS](https://emscripten.org/docs/api_reference/Filesystem-API.html#new-file-system-wasmfs) is "Current Status: Work in Progress".
- [Origin Private File System](https://webkit.org/blog/12257/the-file-system-access-api-with-origin-private-file-system/) was only supported since recently by major browsers.
- [Support of SQLite for WasmFS](https://sqlite.org/wasm/doc/trunk/persistence.md#opfs-wasmfs) was only recently added in version 0.3.43. 

Current browsers [allow to store persistent data (OPFS) in the 3-digit gigabyte range](https://developer.mozilla.org/en-US/docs/Web/API/Storage_API/Storage_quotas_and_eviction_criteria) when there is enough disk space available. This is sufficient for blockchain applications.

### Communication between nodes
Since raw TCP connections are not supported in web browsers, communication can only be implemented over websocket and WebRTC. 

In a first step nodes need to be equipped with websocket communication capabilities. Since browsers cannot act as websocket servers, in-browser nodes will first act as leechers, i.e. they will only connect to public nodes but cannot accept inbound connections. 

Inbound connections could be realized via WebRTC. Later WebRTC might be added to allow nodes accept inbound connections from other in-browser nodes.


### Road to in-browser nodes

- [ ] Rewrite node communication code in an abstract way (using C++'s virtual methods)
- [ ] Add Websocket communication capability to nodes. This requires adding websocket server to listen for incoming websocket connections.
- [ ] Make code modifications to compile node's code using Emscripten. This requires conditionally disabling TCP network libraries and also Websocket server support for webassembly target and conditionally using libraries supporting browser's built websocket support.
- [ ] Figure out a way to extract relevant code from [SQLite's Wasm extension](https://sqlite.org/wasm/doc/trunk/index.md) which is actually a Javascript library. This requires
    - Dissecting [the extension's code](https://sqlite.org/src/dir/ext/wasm) and throwing away all Javascript code to distill the essential things to implant into Warthog codebase.
    - Retrieving relevant compilation flags from the Makefiles.
    - Testing everything with the C++ wrapper for sqlite.
- [ ] Find a way to disallow concurrent access to the database from multiple instances in different tabs. Two nodes should not run against the same database.
- [ ] Mess around with Emscripten peculiarities and additionally required compilation flags (`-sPROXY_TO_PTHREAD`, allow dynamic memory growth, allow shared memories between threads, enabling C++ exceptions etc.).
- [ ] Mess around with browser security (for example special HTTP headers COOP+COEP are required to be sent to support `SharedArrayBuffer`).



