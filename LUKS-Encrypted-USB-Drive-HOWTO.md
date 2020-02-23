# Using a LUKS Encrypted Drive

*This HOWTO is partially based off [this post on vip.asus.com] (https://vip.asus.com/forum/view.aspx?id=20121022225145408&board_id=11&model=RT-N6) from @ryzhovau .*

A USB-equipped ASUS router running Merlin can use [LUKS and cryptsetup](https://gitlab.com/cryptsetup/cryptsetup) to work with an encrypted volume on an attached drive.
(*I have only tested this on my ASUS AC-1900 (a.k.a. RT-AC68P), so I do not know if it will work with other routers.*)

However, the firmware images on the Merlin download repository do not include the necessary kernel modules to do this.  I am too paranoid to install a custom self-built kernel on my router, so I went for a safer approach: build the kernel modules on a separate Linux machine, then copy them over to somewhere on the `/jffs` partition on the router.

### Before You Do This...

Depending on your router model, things might run quite slow--it might not be worth it unless you have a fast router.

This article does not cover auto-mounting the encrypted volume at boot, as a passphrase is required to unlock the volume.  Keeping the password on the router would defeat the entire purpose of encrypting it, so you should find an acceptable way to keep things secure.

The Merlin [`post-mount` user script](https://github.com/RMerl/asuswrt-merlin.ng/wiki/User-scripts) will not try and mount an encrypted volume.  [Here is a post](http://www.snbforums.com/threads/usb-disk-encryption-on-asuswrt-merlin.31586/) describing an alternative way to do it.  (Also refer to the `lmount` and `lumount` scripts below.)

### Steps

1. Download the Merlin source tree:
  ``` 
    # git clone git@github.com:RMerl/asuswrt-merlin.git
  ``` 

  The above will take a little bit of time.  

  By default, the checked out code is the latest stuff, which we do not want.  Instead, it is important that we match the code we checked out with the version installed on the router.  

  Go to the router's admin interface and see what version of the firmware is installed.  On mine, it's `Firmware:380.63_2`.  Luckly, the Merlin sources are tagged by release number, so all we need to do is  
find the tag that matches our installed version:

  ``` 
    # git tag | grep 380.63
    380.63
    380.63-beta1
    380.63_2
  ``` 

  Voila, there it is.  Switch the codebase to the matching release:

  ``` 
    # git checkout 380.63_2 
  ```  

2. Prepare Build Environment: 

  For this, you should [follow the Merlin README.TXT](https://github.com/RMerl/asuswrt-merlin/blob/master/README.TXT) to install dependencies, make required symlinks for the build tools, and update your `PATH`.

3. Configure Kernel Modules:

  At this point, if you followed the README.TXT  from step 2, you should be `cd`'d into the proper `release` directory.  In my case (`RC-AC68P`),  I need to be in `release/src-rt-6.x.4708/`.

  Now we need to update the Linux kernel config file to tell it to build the extra modules we want.  *The actual config file seems to vary by which release folder you are in*, but most likely it's named something like `config_base`.  For the `src-rt-6.x.4708`, the file is `config_base.6a`.  

  Again, this might vary by which release folder you are in, but essentially, you will need to have these entries in the config file:
  ``` 
    CONFIG_MD=y
    CONFIG_BLK_DEV_MD=m
    # CONFIG_MD_LINEAR is not set 
    # CONFIG_MD_RAID0 is not set 
    # CONFIG_MD_RAID1 is not set 
    # CONFIG_MD_RAID10 is not set 
    # CONFIG_MD_RAID456 is not set 
    # CONFIG_MD_MULTIPATH is not set 
    # CONFIG_MD_FAULTY is not set 
    CONFIG_BLK_DEV_DM=m
    # CONFIG_DM_DEBUG is not set
    CONFIG_DM_CRYPT=m
    # CONFIG_DM_SNAPSHOT is not set
    # CONFIG_DM_MIRROR is not set
    # CONFIG_DM_ZERO is not set
    # CONFIG_DM_MULTIPATH is not set
    # CONFIG_DM_DELAY is not set
    # CONFIG_DM_UEVENT is not set


    CONFIG_CRYPTO_XTS=m
    CONFIG_CRYPTO_SHA256=m
  ```

  Here is a patch for the `src-rt-6.x.4708` against the `380.63_2` tag:

  ```
diff --git a/release/src-rt-6.x.4708/linux/linux-2.6.36/config_base.6a b/release/src-rt-6.x.4708/linux/linux-2.6.36/config_base.6a
index 2f6fb71..1267060 100644
--- a/release/src-rt-6.x.4708/linux/linux-2.6.36/config_base.6a
+++ b/release/src-rt-6.x.4708/linux/linux-2.6.36/config_base.6a
@@ -1026,6 +1026,24 @@ CONFIG_SCSI_LOWLEVEL=y
 # CONFIG_SCSI_OSD_INITIATOR is not set
 # CONFIG_ATA is not set
 # CONFIG_MD is not set
+CONFIG_MD=y
+CONFIG_BLK_DEV_MD=m
+# CONFIG_MD_LINEAR is not set
+# CONFIG_MD_RAID0 is not set
+# CONFIG_MD_RAID1 is not set
+# CONFIG_MD_RAID10 is not set
+# CONFIG_MD_RAID456 is not set
+# CONFIG_MD_MULTIPATH is not set
+# CONFIG_MD_FAULTY is not set
+CONFIG_BLK_DEV_DM=m
+# CONFIG_DM_DEBUG is not set
+CONFIG_DM_CRYPT=m
+# CONFIG_DM_SNAPSHOT is not set
+# CONFIG_DM_MIRROR is not set
+# CONFIG_DM_ZERO is not set
+# CONFIG_DM_MULTIPATH is not set
+# CONFIG_DM_DELAY is not set
+# CONFIG_DM_UEVENT is not set
 # CONFIG_FUSION is not set

 #
@@ -1823,7 +1841,7 @@ CONFIG_CRYPTO_CBC=m
 CONFIG_CRYPTO_ECB=y
 # CONFIG_CRYPTO_LRW is not set
 # CONFIG_CRYPTO_PCBC is not set
-# CONFIG_CRYPTO_XTS is not set
+CONFIG_CRYPTO_XTS=m

 #
 # Hash modes
@@ -1845,7 +1863,7 @@ CONFIG_CRYPTO_MD5=y
 # CONFIG_CRYPTO_RMD256 is not set
 # CONFIG_CRYPTO_RMD320 is not set
 CONFIG_CRYPTO_SHA1=y
-# CONFIG_CRYPTO_SHA256 is not set
+CONFIG_CRYPTO_SHA256=m
 # CONFIG_CRYPTO_SHA512 is not set
 # CONFIG_CRYPTO_TGR192 is not set
 # CONFIG_CRYPTO_WP512 is not set
@@ -1921,3 +1939,12 @@ CONFIG_IP_NF_MATCH_ACCOUNT=m
 CONFIG_NET_SCH_CODEL=y
 CONFIG_NET_SCH_FQ_CODEL=y
  ```

  Apply the patch like this:

  ```
  # cd /path/to/your/clone/of/asus-merln
  # patch -p1 -i /path/to/your/luks-merlin.patch
  ```

4. Build Sources

  For this, just follow the README.TXT file to run `make <your_model>`.

  If you run into problems, check out these wiki entries:

    - [Compiling from source using a Debian based Linux Distribution](https://github.com/RMerl/asuswrt-merlin.ng/wiki/Compiling-from-source-using-a-Debian-based-L)
    - [Compile Firmware from source using Ubuntu](https://github.com/RMerl/asuswrt-merlin.ng/wiki/Compile-Firmware-from-source-using-Ubuntu)

  *Note*: there seems to be a way to **just** [build the particular modules we want](http://www.g-loaded.eu/2005/12/20/build-a-single-native-kernel-module/),
but I could not figure out how to do it.  It would be major time saver!

5. Get Compiled Kernel Modules

  Once the build is done, you should have kernel module files that we want to copy to our router.  In particular, these are:

  - In `/<your_build_folder>/linux/linux-2.6.36/crypto`:

  ```
    cbc.ko
    des_generic.ko
    gf128mul.ko
    sha256_generic.ko
    xts.ko
  ```
  - In `/<your_build_folder>/linux/linux-2.6.36/drivers/md`:

  ```
    dm-crypt.ko
    dm-mod.ko
    md-mod.ko
  ```

  Make a folder on the router (I used `/jffs/crypto/modules`), and copy all the `*.ko` files above to that folder.

6. Install Cryptsetup and Test Modules

  All the steps below are done via ssh on the router.

  - Install `cryptsetup` via [entware](https://github.com/Entware/entware):

  ```
    # opkg install cryptsetup
  ```

  - `cd` to where you have your `*.ko` files, and try to load them in this order:
  ```
  # insmod ./dm-mod.ko
  # insmod ./dm-crypt.ko
  # insmod ./sha256_generic.ko
  # insmod ./gf128mul.ko
  # insmod ./xts.ko
  ```

  If no errors were printed when doing all the commands above, you might be set!  You can use `lsmod` to verify they were loaded.

  If you missed one of the modules above, you might get an error on a subsequent one.  For example, trying to `insmod` `xts.ko` before `gf128mul.ko` will fail, because the former depends on the latter.  You can check the output of `dmesg` for hints after a failed `insmod`.  For example, the scenario just described results in the following log output:

  ```
    xts: Unknown symbol gf128mul_x_ble (err 0)
  ```

7. Create Encrypted Volumes

  These procedures are verbatim from `ryzov_al`'s post:

  1. Install cryptsetup and prepare a container file (512Mb in example, current folder is somewhere on USB drive):

  ```
    $ opkg install losetup cryptsetup
    $ dd if=/dev/zero of=./crypto.img bs=1M count=512
    $ insmod ./loop.ko
    $ losetup /dev/loop1 ./crypto.img
  ```

  Now container is mounted as a loop device at /dev/loop1. We have to LUSK-format it:

  ```
    $ opkg install losetup cryptsetup
    $ insmod ./dm-mod.ko
    $ insmod ./aes.ko
    $ insmod ./sha256.ko
    $ insmod ./dm-crypt.ko
    $ cryptsetup -y --key-size 256 luksFormat /dev/loop1

  ```

  2. Mount encrypted container:

  ```
    $ cryptsetup luksOpen /dev/loop1 crypted

  ```

  Now we've got an original block device, so we may use it usual way:

  3. Make a filesystem on encrypted block device:

  ```
    $ mkfs.ext3 -j /dev/mapper/crypted
    $ mkdir /tmp/crypted
    $ mount /dev/mapper/crypted /tmp/crypted/

  ```

  Thats it. We may use /tmp/crypted/ to store encrypted data.

  4. Before shutdown:

  ```
     $ umount /tmp/crypted/
     $ cryptsetup luksClose crypted
     $ losetup -d /dev/loop1
  ```

# Mount Scripts

These scripts assume a convention to facilitate mounting and unmounting of encrypted volumes. 
Both leverage a naming convention: `command foo` means `foo.img` and a mount of `/tmp/mnt/foo`.

# lmount

```
#!/bin/sh

# - Accepts an image base name without the `.img` extension as the sole argument.  
#   For example, given "foo.img", one would run "./lmount foo", then the script will:
#   - Create a loop device if one for this image is not present.
#   - Create `/dev/mapper/foo-crypted` upon successful passphrase entry 
      if the mapper entry is not present.
#   - `insmod` the crypto kernel modules if they are not loaded already. 
#   - Mount the encrypted volume to `/tmp/mnt/foo`.
# - Be sure to set `IMAGE_DIR` were your image(s) are kept on the USB disk.

VOLUME=$1

# where the kernel dm-crypt and related modules live
KMOD_DIR=/jffs/crypto/modules

MOUNT_ROOT=/tmp/mnt

# base directory on the USB disk
# where your encrypted `.img` files are
IMAGE_DIR=$MOUNT_ROOT/images

IMG_FILE="$IMAGE_DIR/$VOLUME.img"
MAPPER_NAME=$VOLUME-crypted      

if [ -z "$VOLUME" ]; then
  echo usage $0 [base_image_name]
  exit 1
fi
                                           
MCOUNT=`cat /proc/mounts | grep -c $MOUNT_ROOT/$VOLUME`
if [ "$MCOUNT" != "0" ]; then
  echo "Volume $VOLUME is already mounted."
  exit 0
fi
                                                     
if [ -f "$IMG_FILE" ]; then
                                                                                  
  # see if there is already a loop device for this image
  local LDEV=`losetup -j $IMG_FILE | sed  's/\(\/dev\/loop[[:digit:]]\+\).\+/\1/'`
                                                       
  if [ -z "$LDEV" ]; then
    LDEV=`losetup -f`
    if [ -z "$LDEV" ]; then
      echo "Could not determine the next free loop device."
      exit 1
    fi
    losetup $LDEV $IMG_FILE
  fi
                                      
  if [ ! -d "/dev/mapper/$MAPPER_NAME" ]; then
    cryptsetup luksOpen $LDEV $MAPPER_NAME
  fi                                                           

  for module in dm-mod dm-crypt sha256_generic gf128mul xts; do
    local mname=`echo $module | sed 's/-/_/g'`
    mcount=`lsmod | grep -c ^${mname}`
    if [ "$mcount" == "0" ]; then
      insmod $KMOD_DIR/${module}.ko
    fi
  done                                                       
                                                    
  [ ! -d "$MOUNT_ROOT/$VOLUME" ] && mkdir $MOUNT_ROOT/$VOLUME
  mount /dev/mapper/$MAPPER_NAME $MOUNT_ROOT/$VOLUME
fi
```



## lumount 

```
#!/bin/sh 

# - Accepts an image base name without the `.img` extension as the sole argument.  
#   For example, given "foo.img", one would run "./lumount foo", then the script will:
#   - `umount` the encrypted volume to `/tmp/mnt/foo`.
#   - Remove (`luksClose`) `/dev/mapper/foo-crypted`.
#   - Remove the loop device for this image.
#   - `rmmod` the crypto kernel modules if no other loop device is present
#   
# - Be sure to set `IMAGE_DIR` were your image(s) are kept on the USB disk.

VOLUME=$1
MOUNT_ROOT=/tmp/mnt
IMAGE_DIR=$MOUNT_ROOT/images
MAPPER_NAME=$VOLUME-crypted

if [ -z "$VOLUME" ]; then
  echo usage $0 [base_image_name]
  exit 1
fi

MCOUNT=`cat /proc/mounts | grep -c $MOUNT_ROOT/$VOLUME`
if [ "$MCOUNT" == "0" ]; then
  echo "Volume $VOLUME is not mounted."
  exit 0
fi

umount $MOUNT_ROOT/$VOLUME

cryptsetup luksClose $MAPPER_NAME

# find loop device
LDEV=`losetup -l -n -O name,back-file | grep $IMAGE_DIR/${VOLUME}.img | awk '{ print $1 }'`

if [ -n "$LDEV" ]; then
  echo "Removing loop device: $LDEV"
  losetup -d $LDEV
else
  echo "Could not find loop device for volume ${VOLUME}"
  exit 1
fi

# remove kernel modules if no other loop dev is around 
LDEV_COUNT=`losetup -l -n | wc -l`

if [ "$LDEV_COUNT" == "0" ]; then 
  echo "Removing crypto kernel modules"
  
  for module in xts gf128mul sha256_generic dm-crypt dm-mod; do
    rmmod $module
  done
fi
```


# References

https://wiki.gentoo.org/wiki/Dm-crypt#Kernel_Configuration
https://vip.asus.com/forum/view.aspx?id=20121022225145408&board_id=11&model=RT-N66U+

http://www.g-loaded.eu/2005/12/20/build-a-single-native-kernel-module/
https://github.com/assarbad/build-asuswrt-merlin

https://github.com/RMerl/asuswrt-merlin/issues/328
https://github.com/RMerl/asuswrt-merlin/issues/285

http://www.snbforums.com/threads/compiling-kernel-module.11790/