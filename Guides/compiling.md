---
label: Compiling from Source
---
# Compiling Warthog from source on Linux
Compiling Warthog from soure is an alternative to the use precompiled binaries. 

## Installing required packages
Before we can start make sure you have a recent Linux distribution. In this guide we are using Ubuntu 22.04.3 LTS. We need to update our package manager and install `git`, `build-essential`, `meson` and `ninja-build`:
```
sudo apt update
sudo apt install git
sudo apt install build-essential meson ninja-build
```
![](/img/get-started/01-apt-update.png)

![](/img/get-started/02-install-git.png)
![](/img/get-started/03-install-build.png)


## Cloning the Repo
Now that we have git we can clone the Warthog repository from GitHub:
```
git clone https://github.com/warthog-network/Warthog
```
![](/img/get-started/04-clone.png)

## Compiling
A new directory should be created, let's `cd` into it and run `meson build` to create a build directory named `build`:
```
cd Warthog
meson build
```
![](/img/get-started/05-meson-build.png)

When you run the `meson build` command for the first time some extra dependencies will be downloaded so it might take a while.  Now we have a build directory within the Warthog directory so we `cd` into it and start compilation with `ninja`:
```
cd build
ninja
```
![](/img/get-started/06-ninja.png)

Congratulations! You now have compiled the Warthog C++ source. But wait - there is a problem: we did not enable compiler optimizations. In `meson` compiler optimizations need to be explicitly enabled with the `--buildtype=release` flag. Then the compiled executables and libraries will be more efficient. This is important for mining because you will get better hashrate with optimized compilation. 

So let's do this again, we will go up one directory and delete the build directory again. Then we recreate it but this time with the `--buildtype=release` option and compile again with `ninja`:

```
cd ..
rm -rf build
meson build --buildtype=release
cd build
ninja
```
Note that now meson reports it has set `buildtype: release`.
![](/img/get-started/07-ninja-release.png)

After compilation finished successfully we can have a look at the compiled artifacts: Warthog currently compiles a node, a wallet and a miner. They are generated in the `src` directory within the build directory. Have a look:
```
cd src/
ls
```
![](/img/get-started/08-ls-compiled.png)

