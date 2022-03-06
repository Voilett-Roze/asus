# How to build Asuswrt-merlin

> **Status**:
> The current recommended build system is Ubuntu 20.04 LTS, and this documentation
> has been adapted to that.  Documentation for building on other Linux
> flavors has been retained but is likely obsolete and will be marked as
> such.

## Setup a virtual machine for builds

### *Windows 10 (version 2004 and newer)*

On Windows 10 version 2004 and newer, the easiest way to set up a virtual machine for builds
is by using the Windows Subsystem for Linux (WSL2).  See [Setting up Build VM under WLS2](Setting-up-Build-VM-under-WSL2)
for instructions on how to configure WSL for a build VM.

### *Linux, MacOS, older Windows*

On Linux systems, MacOS and older Windows versions, the easiest way to set up a virtual machine
for builds is by using Canonical's Multipass.  See [Setting up Build VM under Multipass](Setting-up-Build-VM-under-Multipass)

### *Native builds on other Linux flavors*

It is highly recommended to use Multipass to create a build VM instead of trying to build
natively on your Linux system.  However, for those who are interested in building natively
on your Linux system, it is best to follow the steps below (adapted for your specific flavor
of Linux) instead of the ***OBSOLETE*** documentation found elsewhere in this Wiki.

## Setting up the Linux environment

The rest will be identical to what you would do in any actual Linux VM: get a copy of the toolchain, set it up,
then get a copy of the source code, and get building.

**Make sure that you do all of this inside the instance.**

First, you need to enable support for 32-bit libraries, required by the toolchain.  To do so:

```bash
sudo dpkg --add-architecture i386
```

You also need to switch from dash to bash as the default shell (/bin/sh).  This is required by the newer build system used by Broadcom for the HND-based models.  To do so, run the following command, and when asked, tell it NOT to use Dash:

```bash
sudo dpkg-reconfigure dash
```

Now you can install all the required packages.  Note that this list may change over time as new components are added to the firmware code.

```bash
sudo apt update
sudo apt-get install libtool-bin cmake libproxy-dev uuid-dev liblzo2-dev autoconf automake bash bison \
bzip2 diffutils file flex m4 g++ gawk groff-base libncurses5-dev libtool libslang2 make patch perl pkg-config shtool \
subversion tar texinfo zlib1g zlib1g-dev git gettext libexpat1-dev libssl-dev cvs gperf unzip \
python libxml-parser-perl gcc-multilib gconf-editor libxml2-dev g++-multilib gitk libncurses5 mtd-utils \
libncurses5-dev libvorbis-dev git autopoint autogen automake-1.15 sed build-essential intltool libglib2.0-dev \
xutils-dev lib32z1-dev lib32stdc++6 xsltproc gtk-doc-tools libelf1:i386
```

Now we are going to download RMerlin's hard work.

> **Note**:
> A very detailed guide on how to download the source code can be found [on this Wiki page](/RMerl/asuswrt-merlin.ng/wiki/Download-the-latest-source-code-from-GitHub).

```bash
cd $HOME
git clone https://github.com/RMerl/am-toolchains
git clone https://github.com/RMerl/asuswrt-merlin.ng
```

It's recommended not to compile directly from your cloned repo, but instead to work on a copy of it.  This allows you to easily clean things up simply by re-syncing it with your clean copy.

```bash
mkdir amng-build
rsync -a --del asuswrt-merlin.ng/ amng-build
```

You will now have to setup your toolchain.

**Broadcom HND ARM platform (RT-AC86U, GT-AC2900):**

```bash
sudo ln -s ~/am-toolchains/brcm-arm-hnd /opt/toolchains
echo "export LD_LIBRARY_PATH=\$LD_LIBRARY:/opt/toolchains/crosstools-arm-gcc-5.3-linux-4.1-glibc-2.22-binutils-2.25/usr/lib" >> ~/.profile
echo "export TOOLCHAIN_BASE=/opt/toolchains" >> ~/.profile
echo "PATH=\$PATH:/opt/toolchains/crosstools-arm-gcc-5.3-linux-4.1-glibc-2.22-binutils-2.25/usr/bin" >> ~/.profile
echo "PATH=\$PATH:/opt/toolchains/crosstools-aarch64-gcc-5.3-linux-4.1-glibc-2.22-binutils-2.25/usr/bin" >> ~/.profile
```

**Broadcom HND AX ARM platform (RT-AX56U, RT-AX58U, RT-AX86U, RT-AX88U, GT-AX11000, RT-AX68U):**

```bash
sudo ln -s ~/am-toolchains/brcm-arm-hnd /opt/toolchains
echo "export LD_LIBRARY_PATH=\$LD_LIBRARY:/opt/toolchains/crosstools-arm-gcc-5.5-linux-4.1-glibc-2.26-binutils-2.28.1/usr/lib" >> ~/.profile
echo "export TOOLCHAIN_BASE=/opt/toolchains" >> ~/.profile
echo "PATH=\$PATH:/opt/toolchains/crosstools-arm-gcc-5.5-linux-4.1-glibc-2.26-binutils-2.28.1/usr/bin" >> ~/.profile
echo "PATH=\$PATH:/opt/toolchains/crosstools-aarch64-gcc-5.5-linux-4.1-glibc-2.26-binutils-2.28.1/usr/bin" >> ~/.profile
```

**Broadcom SDK6/SDK7 ARM platform (RT-AC68U, RT-AC88U, RT-AC3100, RT-AC5300):**

```bash
sudo ln -s ~/am-toolchains/brcm-arm-sdk/hndtools-arm-linux-2.6.36-uclibc-4.5.3 /opt/brcm-arm
ln -s ~/am-toolchains/brcm-arm-sdk  ~/amng-build/release/src-rt-6.x.4708/toolchains
echo "PATH=\$PATH:/opt/brcm-arm/bin" >> ~/.profile
```

Your setup is now ready for compiling, from inside the `amng-build` folder.

**RT-AC68U firmware**
```bash
cd ~/amng-build/release/src-rt-6.x.4708
make rt-ac68u
```

**RT-AC88U firmware**
```bash
cd ~/amng-build/release/src-rt-7.14.114.x/src
make rt-ac88u
```

**RT-AC3100 firmware**
```bash
cd ~/amng-build/release/src-rt-7.14.114.x/src
make rt-ac3100
```

**RT-AC5300 firmware**
```bash
cd ~/amng-build/release/src-rt-7.14.114.x/src
make rt-ac5300
```

**RT-AC86U firmware**
```bash
cd ~/amng-build/release/src-rt-5.02hnd
make rt-ac86u
```

**RT-AX56U firmware**
```bash
cd ~/amng-build/release/src-rt-5.02axhnd.675x
make rt-ax56u
```

**RT-AX58U firmware**
```bash
cd ~/amng-build/release/src-rt-5.02axhnd.675x
make rt-ax58u
```

**RT-AX86U firmware**
```bash
cd ~/amng-build/release/src-rt-5.02p1axhnd.675x
make rt-ax86u
```

**RT-AX88U firmware**
```bash
cd ~/amng-build/rrelease/src-rt-5.02axhnd
make rt-ax88u
```

**GT-AC2900 firmware**
```bash
cd ~/amng-build/release/src-rt-5.02hnd
make gt-ac2900
```

**GT-AX11000 firmware**
```bash
cd ~/amng-build/release/src-rt-5.02axhnd
make gt-ax11000
```

**RT-AX68U firmware**
```bash
cd ~/amng-build/release/src-rt-5.02p1axhnd.675x
make rt-ax68u
```

## References

1. [Asuswrt-merlin build script](https://github.com/RMerl/asuswrt-merlin.ng/blob/master/tools/build-all)
2. [Asuswrt-merlin toolchains](https://github.com/RMerl/am-toolchains)

## Useful links

1. [Asuswrt-merlin - Custom Firmware Manager](https://github.com/Adamm00/amcfwm)
