### Introduction
Starting with Windows 10 version 2004, Microsoft greatly enhanced the built-in Windows Subsystem for Linux (WSL).  WSL2 now uses an actual Linux kernel, which allows, among other things, to use an ext4 filesystem, and also greatly enhance performance.

In testing, WSL2 even ended up being faster than a Virtualbox VM at compiling a firmware (30 minutes for VBox versus 25 mins for WSL2, tested on a laptop using an AMD 4500U).  This makes WSL2 a very environment for doing firmware builds.


### Setup WSL2
First, you need to be running Windows 10 version 2004 or newer (released in May 2020).  Once you are, you need to enable WSL2, install a supported Linux distro (I recommend Ubuntu 18.04 or 20.04, the latter hasn't been fully tested with Asuswrt-Merlin yet), and make sure that this distro is set in WSL2 mode (it probably defaults to WSL1).

For details, please refer to the Microsoft documentation:

https://docs.microsoft.com/en-us/windows/wsl/install-win10

Also, you might want to limit the number of CPU cores or memory that WSL2 is allowed to use.  This can be configured through the .wlsconf config file.  See the "Configure Global Options" section:

https://docs.microsoft.com/en-us/windows/wsl/wsl-config


### Setting up the Linux environment
You need to configure WSL to NOT insert the Windows search path within the Linux environment's own search path.  Failing to do so will greatly slow down the build process (in tests, the build time increased from taking 30 minutes, to taking 60-90 minutes).  Edit (or create) /etc/wsl.conf, and add the following:

```
[interop]
appendWindowsPath=false
```
Next, restart the WSL instance.  First log out of it (by typing "exit"), then open a Windows command prompt to issue the following command (edit the instance label to use whatever you used, if it's not ubuntu-18.04)

```
wsl --shutdown Ubuntu-18.04
```

You can see the actual label name by viewing the list of installed WSL instances:

```
wsl --list 
```

Then restart your WSL instance by re-launching Ubuntu.

Now, you need to enable support for 32-bit libraries, required by the toolchain.  To do so:

```
sudo dpkg --add-architecture i386
```

You also need to switch from dash to bash as the default shell (/bin/sh).  This is required by the newer build system used by Broadcom for the HND-based models.  To do so, run the following command, and when asked, tell it NOT to use Dash:

```
sudo dpkg-reconfigure dash
```

Now you can install all the required packages.  Note that this list may change over time as new components are added to the firmware code.

```
sudo apt-get install libtool-bin cmake libproxy-dev uuid-dev liblzo2-dev autoconf automake bash bison \
bzip2 diffutils file flex m4 g++ gawk groff-base libncurses5-dev libtool libslang2 make patch perl pkg-config shtool \
subversion tar texinfo zlib1g zlib1g-dev git gettext libexpat1-dev libssl-dev cvs gperf unzip \
python libxml-parser-perl gcc-multilib gconf-editor libxml2-dev g++-multilib gitk libncurses5 mtd-utils \
libncurses5-dev libvorbis-dev git autopoint autogen sed build-essential intltool libglib2.0-dev \
xutils-dev lib32z1-dev lib32stdc++6 xsltproc gtk-doc-tools libelf1:i386
```

Ubuntu 20.04 will probably require this as well:

```
sudo apt install automake-1.15
```

### Setting up build environment
The rest will be identical to what you would do in any actual Linux VM: get a copy of the toolchain, set it up, then get a copy of the source code, and get building.  Make sure that you do all of this inside your Linux home folder, NOT inside the Windows path.  Your home folder will be in a native ext4 filesystem.

```
cd $HOME
git clone https://github.com/RMerl/am-toolchains
git clone https://github.com/RMerl/asuswrt-merlin.ng
```

It's recommended not to compile directly from your cloned repo, but instead to work on a copy of it.  This allows you to easily clean things up simply by re-syncing it with your clean copy.

```
mkdir amng-build
rsync -a --del asuswrt-merlin.ng/ amng-build
```

You will now have to setup your toolchain.

```
sudo ln -s ~/am-toolchains/brcm-arm-hnd /opt/toolchains

echo "export LD_LIBRARY_PATH=$LD_LIBRARY:/opt/toolchains/crosstools-arm-gcc-5.3-linux-4.1-glibc-2.22-binutils-2.25/usr/lib" >> ~/.profile
echo "export TOOLCHAIN_BASE=/opt/toolchains" >> ~/.profile
echo "PATH=\$PATH:/opt/toolchains/crosstools-arm-gcc-5.3-linux-4.1-glibc-2.22-binutils-2.25/usr/bin" >> ~/.profile
echo "PATH=\$PATH:/opt/toolchains/crosstools-aarch64-gcc-5.3-linux-4.1-glibc-2.22-binutils-2.25/usr/bin" >> ~/.profile

echo "export LD_LIBRARY_PATH=$LD_LIBRARY:/opt/toolchains/crosstools-arm-gcc-5.5-linux-4.1-glibc-2.26-binutils-2.28.1/usr/lib" >> ~/.profile
echo "PATH=\$PATH:/opt/toolchains/crosstools-arm-gcc-5.5-linux-4.1-glibc-2.26-binutils-2.28.1/usr/bin" >> ~/.profile
echo "PATH=\$PATH:/opt/toolchains/crosstools-aarch64-gcc-5.5-linux-4.1-glibc-2.26-binutils-2.28.1/usr/bin" >> ~/.profile

sudo ln -s ~/am-toolchains/brcm-arm-sdk/hndtools-arm-linux-2.6.36-uclibc-4.5.3 /opt/brcm-arm
ln -s ~/am-toolchains/brcm-arm-sdk  ~/asuswrt-merlin.ng/release/src-rt-6.x.4708/toolchains
echo "PATH=\$PATH:/opt/brcm-arm/bin" >> ~/.profile
```

Your setup is now ready for compiling, from inside the amng-build folder.  Please refer to the Wiki for more information as how to compile specific router models.
