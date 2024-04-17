# HOWTO Run a warthog node with docker.

## Basic way

```sh
docker run zzjulien/warthog_node:latest
```

## Modify your node

Exemple you can add this type of arguments to run a node with stratum enabled

```sh
docker run --expose 3456 zzjulien/warthog_node:latest --stratum=0.0.0.0:3456
```
## Run a public node

```sh
docker run --expose 3001 zzjulien/warthog_node:latest --enable-public
```
