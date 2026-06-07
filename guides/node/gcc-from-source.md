# Installing GCC from source
In case your system packages provide a GCC version that is too old (< version 11) you can install a more modern GCC version from source. This guide is for linux.

After checking the [prerequisites](https://gcc.gnu.org/install/prerequisites.html) you can download the source code from a [mirror site](https://gcc.gnu.org/mirrors.html), like [this one](https://mirrorservice.org/sites/sourceware.org/pub/gcc/releases/). 
You can also check the gpg signature. To this end you first need to have the keyring imported:

```
curl http://ftp.gnu.org/gnu/gnu-keyring.gpg -O
gpg --import gnu-keyring.gpg
```

Select a recent version, download and unzip it.
![](/img/gcc-select.png)

```
tar -xf gcc-16.1.0.tar.gz 
cd gcc-16.1.0
```
Next we need to download the prerequisites by invoking an automated download script:
```
./contrib/download_prerequisites
```
After this is done we can configure the build and call `make`

```
./configure
make -j16
```
Above, the `-j16` flags specifies that 16 jobs can run in parallel. Adapt this number to your system to maximize the parrallelism. The build takes a long time.
After the build has succeeded you can install gcc as a privileged user.







