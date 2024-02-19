# OpenCL Installation
To mine on your GPUs you need to install GPU drivers with OpenCL support.

You can check whether your GPU has a working OpenCL support with the `clinfo` command. Install this program with 
```
sudo apt install clinfo
```
Then start it by typing
```
clinfo
```
In our case we don't have any GPU drivers installed that support OpenCL:

![](/img/get-started/clinfo.png)

The installation procedure depends on your GPU vendor. 

### AMD GPUs

Our GPU is an AMD Radeon RX 5600 XT but `clinfo` does not detect any OpenCL installation for it yet. Visit the AMD support site [https://www.amd.com/en/support]() and select the GPU model:

![](/img/get-started/amd-support.png)

![](/img/get-started/amd-download.png)

Then we need to install the downloaded `deb` package.

![](/img/get-started/amd-install-package.png)

![](/img/get-started/amd-installed.png)

```
sudo amdgpu-install --usecase=workstation,rocm,opencl --opencl=rocr,legacy --vulkan=pro --accept-eula
```

This will command will take a while downloading and installing more than 2GB worth of packages.

Now try `clinfo` agian:

![](/img/get-started/clinfo-no-devices.png)

The AMD OpenCL platform is now detected but there are no devices. This is because you need to add your user to the `render` group:

```
sudo adduser $USER render
```
Now reboot your system. If you try `clinfo` again you now see the GPU device:

![](/img/get-started/clinfo-success.png)


### NVIDIA GPUs
First download your driver from [https://www.nvidia.de/Download/index.aspx?lang=en-us]() by selecting your GPU model and clicking on "Aggree Download".
![](/img/get-started/nvidia-download.png)

![](/img/get-started/nvidia-confirm.png)

The downloaded file is a `sh` script and we need to make it executable. So we navigate to the `Downloads` directory and execute 
```
chmod a+x NVIDIA-Linux-x86_64-535.112.01.run
```
Of course you need to insert your exact filename here.
![](/img/get-started/nvidia-downloaded.png)
When we try to run it via

```
sudo ./NVIDIA-Linux-x86_64-535.112.01.run
```
we get this error:
![](/img/get-started/nvidia-failed-install.png)

This means we need to execute the script no a lower run level without an X server running. In addition we must blacklist the default `nouvaeu` driver with this command:
```
echo -e "blacklist nouveau\noptions nouveau modeset=0" | sudo tee /etc/modprobe.d/blacklist-nouveau.conf
```
Now reboot your system. Once rebooted you should switch to a lower run level.
To do this press the three keys CTRL + ALT + F3 at the same time.
You should see a black screen with login prompt like this:
```
Ubuntu 220.4.3 LTS pc tty3

pc login: _
```
Log in with your user name and password and then switch to a lower run level with 
```
sudo init 3
```
You can now start the installation as above, navigate to the `Downloads` directory and start the installation:
```
cd ~/Downloads
sudo ./NVIDIA-Linux-x86_64-535.112.01.run
```

You should see these screens, just select "Confirm installation", then "Yes", "Yes", "OK".


![](/img/get-started/nvidia-continue-install.png)

![](/img/get-started/nvidia-compatibility.png)


![](/img/get-started/nvidia-x-config.png)

![](/img/get-started/nvidia-complete.png)

Now that the NVIDIA driver installation is complete restart your system again you should be able to see your device in the `clinfo` command:
![](/img/get-started/clinfo-success-nvidia.png)
