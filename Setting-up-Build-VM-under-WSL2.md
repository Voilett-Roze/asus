### Introduction
Starting with Windows 10 version 2004, Microsoft greatly enhanced the built-in Windows Subsystem for Linux (WSL).  WSL2 now uses an actual Linux kernel, which allows, among other things, to use an ext4 filesystem, and also greatly enhance performance.

In testing, WSL2 even ended up being faster than a Virtualbox VM at compiling a firmware (30 minutes for VBox versus 25 mins for WSL2, tested on a laptop using an AMD Ryzen 5 4500U).  This makes WSL2 a very environment for doing firmware builds.


### Setup WSL2
First, you need to be running Windows 10 version 2004 or newer (released in May 2020).  Once you are, you need to enable WSL2, install a supported Linux distro (I recommend Ubuntu 18.04 or 20.04, the latter hasn't been fully tested with Asuswrt-Merlin yet), and make sure that this distro is set in WSL2 mode (it probably defaults to WSL1).

For details, please refer to the Microsoft documentation:

https://docs.microsoft.com/en-us/windows/wsl/install-win10

Also, you might want to limit the number of CPU cores or memory that WSL2 is allowed to use.  This can be configured through the .wlsconf config file.  See the "Configure Global Options" section:

https://docs.microsoft.com/en-us/windows/wsl/wsl-config


### Setting up the Linux environment
You need to configure WSL to NOT insert the Windows search path within the Linux environment's own search path.  Failing to do so will greatly slow down the build process (in tests, the build time increased from taking 30 minutes to taking 60-90 minutes).  Edit (or create) /etc/wsl.conf, and add the following:

```
[interop]
appendWindowsPath=false
```
Next, restart the WSL instance.  First log out of it (by typing "exit"), then open a Windows command prompt to issue the following command (edit the instance label to use whatever you used, if it's not ubuntu-20.04)

```
wsl --shutdown Ubuntu-20.04
```

You can see the actual label name by viewing the list of installed WSL instances:

```
wsl --list
```

Then restart your WSL instance by re-launching Ubuntu.
