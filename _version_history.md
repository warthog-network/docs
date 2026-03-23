# Version History

* v0.4.25 : Core developers have started working on the network part
* **v0.4.24** (last stable version): Add Synced field for two endpoint ()
* v0.4.22 : New endpoint for pool development
* v0.4.21 : Disable mine/ endpoint for old structure block.
* v0.4.20 : delete cached stratum block when chain advances
* v0.4.19 : Fix of a solo stratum bug
* v0.4.18 : Add additional info in an endpoint + more error code
* v0.4.17 : Add markle prefix in an endpoint
* v0.4.16 : Finish the websocket endpoint
* v0.4.15 : disable transactions as fallback
* v0.4.14 : Add wallet tools in API endpoint
* v0.4.13 : Fix the json key for the previously added API method 
* v0.4.12 : Modification of the API to allow you to modify the number of blocks on which you want a hashrate hestimation.
* v0.4.11 : Rework of the block generation (blank when there is 0 transfers)
* v0.4.10 : fix reward section of the block generation
* v0.4.9 : fix transfer count in the block generation part of the code + disable interactive mode of the wallet when all parameters are given
* v0.4.8 : rework of transaction propagation
* v0.4.7 : fix an bug in blog generation
* v0.4.6 : enable macos build
* v0.4.5: Solo Stratum pool integrated into node, use `--stratum=0.0.0.0:3456` to enable on port `3456`
* v0.4.1-2: increase max transactions mined into blocks + mempool sorted by fee
* v0.4.0: fix mempool bug
