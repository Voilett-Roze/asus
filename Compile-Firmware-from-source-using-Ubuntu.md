# How to build asuswrt-merlin on Ubuntu
.
.
.
.
### If wanting to build for the new HND platform (RT-AC86U), some different instructions have been added and need to be followed. ###
.
.
.
.

## Cautionary note

> WARNING DESPITE THE FACT THAT IT IS POSSIBLE TO COMPILE YOUR OWN FIRMWARE,
> I DO NOT ADVISE YOU TO DO SO
> THIS GUIDE IS NOT WRITTEN BY A PROFESSIONAL BUT BY AN AMATEUR<>NOVICE
> THERE ARE A LOT OF THINGS THAT COULD GO WRONG DURING THE PROCESS
> THE AUTHOR OF THIS GUIDE NOR THE SOURCE PROVIDER CAN NOT BE HELD RESPONSIBLE FOR ANY DAMAGE THAT MAY OCCUR BY 
> FLASHING YOUR SELF COMPILED FIRMWARE ON YOUR ROUTER
> IF YOU REALLY WANT TO LEARN THIS TYPE OF COMPUTING AND CODING I SUGGEST YOU FIRST GET FAMILIAR WITH LINUX

But on the other hand your router is VERY hard to brick!

## Additional information

Ubuntu is a derivative of Debian, so you may benefit from also looking at [the instructions for Debian-derived distributions in general](/RMerl/asuswrt-merlin/wiki/Compiling-from-source-using-a-Debian-based-Linux-Distribution).

## Example: building under Ubuntu 12.04

For the reference we are going to use Ubuntu 12.04 in VMware Player.

* You can download Ubuntu [here](http://www.ubuntu.com/download)
* And VMware Player [here](https://my.vmware.com/web/vmware/free#desktop_end_user_computing/vmware_player/5_0)

When you have both install VMware Player.

Follow these steps :

* Create a New Virtual Machine

![Create](http://members.home.nl/frits.pruymboom/Howto%20Compile%20From%20Source/Create%20new%20virtual%20mashine.png)

* Select Installer disc image file (iso) ( VMware Player will tell you something about easy install, that's okay )

![Select Iso](http://members.home.nl/frits.pruymboom/Howto%20Compile%20From%20Source/installer%20disc%20iso.png)

* When installing Ubuntu it asks for a username it's best if you just name it ''router'' without the quotes because then you can just copy and paste from this guide no need to make everything complicated.

![Name](http://members.home.nl/frits.pruymboom/Howto%20Compile%20From%20Source/important%20name.png)

* Give it about 20 GB afterwards you can delete your virtual Ubuntu anyway.

![20gb](http://members.home.nl/frits.pruymboom/Howto%20Compile%20From%20Source/20gb.png)

* Crank up some specs give it some more ram and procesing power, It doesn't sound as a heavy work load but building your image for the router actually takes a long time on a slow processor

![customize](http://members.home.nl/frits.pruymboom/Howto%20Compile%20From%20Source/customiza%20hardware.png)

![crank](http://members.home.nl/frits.pruymboom/Howto%20Compile%20From%20Source/crank%20up%20the%20specs.png)

When the Ubuntu installer has finished you can fire up terminal

* Fire up a terminal CTRL+ALT+T

## Manually preparing the build environment

* We are going to download some packages ( I am sure there are some you don't need, but this is how I got it working and you can delete Ubuntu afterward so who cares right ? )

Paste in:

```bash
sudo apt-get install libtool-bin cmake libproxy-dev uuid-dev liblzo2-dev autoconf automake bash bison /
bzip2 diffutils file flex m4 g++ gawk groff-base libncurses-dev libtool libslang2 make patch perl pkg-config shtool /
subversion tar texinfo zlib1g zlib1g-dev git-core gettext libexpat1-dev libssl-dev cvs gperf unzip /
python libxml-parser-perl gcc-multilib gconf-editor libxml2-dev g++-multilib gitk libncurses5 mtd-utils /
libncurses5-dev libvorbis-dev git autopoint autogen sed build-essential intltool libelf1:i386 libglib2.0-dev /
xutils-dev lib32z1-dev lib32stdc++6
```
If you have Ubuntu x64 edition then you need `lib32z1-dev` and `lib32stdc++6`

```bash
sudo apt-get install lib32z1-dev lib32stdc++6
```
If you are ready, install these packages & make yourself a well deserved coffee or have a cold beer.

Now we are going to download RMerlin's hard work.

**Note:** a very detailed guide on how to download the source code can be found [on this Wiki page](/RMerl/asuswrt-merlin/wiki/Download-the-latest-source-code-from-GitHub).

Legacy 380.xx branch

```bash
git clone https://github.com/RMerl/asuswrt-merlin
```

Next Generation branch for the HND platform (RT-AC86u)

```bash
git clone https://github.com/RMerl/asuswrt-merlin.ng
```

Also you'll need the toolchains

```bash
sudo ln -s ~/am-toolchains/brcm-arm-hnd /opt/toolchains
```

With these command you will build your environment which you will need to work with.  Also at this point, if you wish to build for the HND platform you'll need to scroll on and follow some slightly different instructions.

```bash
sudo ln -s ~/asuswrt-merlin/tools/brcm /opt/brcm
sudo ln -s ~/asuswrt-merlin/release/src-rt-6.x.4708/toolchains/hndtools-arm-linux-2.6.36-uclibc-4.5.3 /opt/brcm-arm
```

Adjust the `PATH` environment variable:

```bash
export PATH=$PATH:/opt/brcm/hndtools-mipsel-linux/bin:/opt/brcm/hndtools-mipsel-uclibc/bin:/opt/brcm-arm/bin
```

For the new HND platform (RT-AC86u):

```bash
sudo ln -sf bash /bin/sh

echo "export LD_LIBRARY_PATH=$LD_LIBRARY:/opt/toolchains/crosstools-arm-gcc-5.3-linux-4.1-glibc-2.22-binutils-2.25/usr/lib" >> ~/.profile

echo "export TOOLCHAIN_BASE=/opt/toolchains" >> ~/.profile

echo "PATH=\$PATH:/opt/toolchains/crosstools-arm-gcc-5.3-linux-4.1-glibc-2.22-binutils-2.25/usr/bin" >> ~/.profile

echo "PATH=\$PATH:/opt/toolchains/crosstools-aarch64-gcc-5.3-linux-4.1-glibc-2.22-binutils-2.25/usr/bin" >> ~/.profile
```

Set a few more symlinks:

```bash
sudo mkdir -p /media/ASUSWRT/
```

```bash
sudo ln -s ~/asuswrt-merlin /media/ASUSWRT/asuswrt-merlin
```

For the new HND platform (RT-AC86u):

```bash
sudo ln -s ~/asuswrt-merlin.ng /media/ASUSWRT/asuswrt-merlin.ng
```

## Ready to build: manual approach

### RT-N16

```
cd ~/asuswrt-merlin/release/src-rt
make clean
make rt-n16
```

### RT-N66U & RT-AC66U

```
cd ~/asuswrt-merlin/release/src-rt-6.x
make clean
```

#### ... then for RT-N66U

```
make rt-n66u
```

#### ... and for RT-AC66U

```bash
make rt-ac66u
```

### RT-AC56U & RT-AC68U

```bash
cd ~/asuswrt-merlin/release/src-rt-6.x.4708
make clean
```

#### ... then for RT-AC56U

```bash
make rt-ac56u
```
#### ... and for RT-AC68U

```bash
make rt-ac68u
```


#### for RT-AC86U

```bash
cd ~/asuswrt-merlin.ng/release/src-rt-5.02hnd
make rt-ac86u
```

## Notes for Ubuntu 13.10 and later

If you want to build in Ubuntu 13.10, before you `make clean` / `make [router]`, you might need to perform these steps due to the different version of `autoconf`.

This works to build 3.0.0.4_374.38_1.

NOTE: **For the new HND platform (RT-AC86u) routes will need to be changed accordingly, where asuswrt-merlin is stated chenge to asuswrt-merlin.ng

```bash
sudo apt-get install libproxy-dev

# fix neon missing proxy.h
cp /usr/include/proxy.h ~/asuswrt-merlin/release/src/router/neon/

# fix broken configure script for libdaemon
cd ~/asuswrt-merlin/release/src/router/libdaemon
aclocal

# fix broken configure script for libxml2
cd ~/asuswrt-merlin/release/src/router/libxml2
sed -i s/AM_C_PROTOTYPES/#AM_C_PROTOTYPES/g ~/asuswrt-merlin/release/src/router/libxml2/configure.in
aclocal

# fix broken configure script for libvorbis
cd ~/asuswrt-merlin/release/src/router/libvorbis
aclocal
```

## Automated all-in-one script

**Status:** This has been tested on Ubuntu 12.04 and 14.04 and is known to work as of 2015-02-02.

The latest version can always be found at [assarbad/build-asuswrt-merlin](https://github.com/assarbad/build-asuswrt-merlin). Please report issues there as well.

On the landing page (above link) you will find a brief introduction on how to use the script as well as its status.