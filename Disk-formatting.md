# Guide to disk formatting with ASUS-WRT Merlin

Learn how ASUS routers can be used to directly repartition and format attached USB disks from the command line. Follow a step-by-step guide or use a simple interactive menu-based script to automatically format disks to ext2, ext3, ext4 filesystem.

## What you'll learn in this guide:
- Limitations of managing disks directly on the router
- [Automatically formatting disks with the AMTM script](#automatic-disk-formatting-script)
- Wipe/Zero disk with dd to avoid errors caused by old partition tables
- Create a new partition table
- Create a new partition
- Format a partition to ext2, ext3, ext4
- Adjust filesystem features like disk labels and journaling
- Disk mounting and unmounting
---- 
## Contents:
- About
	- [Requirements](#requirements)
	- [Limitations & Important Notes](#limitations--important-notes)
	- [Methods: Automatic or Manual?](#methods-automatic-or-manual)
- [Automatic Disk Formatting Script](#automatic-disk-formatting-script)
- [Manual Step-by-Step Instructions](#manual-step-by-step-instructions)
	1. [Preparations](#1-preparations)
	2. [Attach disk](#2-attach-usb-disk)
	3. [Connect to router via ssh](#3-connect-to-router-via-ssh)
	4. [View disk information](#4-view-disk-information)
	5. [Unmount](#5-unmount)
	6. [Zero disk](#6-zero-disk)
	7. [Repartition disk](#7-repartition-disk)
	8. [Format and adjust filesystem features](#8-format-and-adjust-filesystem-features)
	9. [Reboot and mount disk](#9-reboot-and-mount-disk)
- Thanks
---- 
## About

### Requirements
- USB Storage Device cannot exceed 2TB disk capacity.
- Physical access to attach and detach disks to router USB port.
- Basic command line skills.
----
### Limitations & Important Notes

#### Partition tables
There are some compatibility issues with partition tables:
- **MBR** _'Master Boot Record'_ has best compatibility with ASUS-WRT
- **MBR** supports maximum disk capacity 2TB (entire disk)
- **GPT** _'GUID Partition Table'_ supports disks over 2TB
- **GPT** is unfortunately not supported by **fdisk** on ASUS-WRT so devices using GPT will appear locked in **fdisk** and may return error message similar to "Found valid GPT with protective MBR; using GPT". In this case the existing GPT must be erased without fdisk. One solution is to zero the disk. Another solution is the disk formatting feature in AMTM.
	> Quote: "Master Boot Record partition table limits the maximum size of the entire disk (not just a partition) to 2TB. If you have a disk larger than this you need to use a GUID partition table (GPT). Whilst personally I always use MBR if possible for compatibility reasons, we know that Asus officially supports disks of at least 4TB, ergo they must also support GPT. But here's the rub, the router's version of fdisk doesn't support GTP partition tables let alone have the option to create them. So owners of such devices ... will have to partition them with GTP on another device and then skip the whole of that step ..." -- @ColinTaylor [Source](https://www.snbforums.com/threads/ext4-disk-formatting-options-on-the-router.48302/page-2#post-456414)

#### Unmounting disks
There are a few problems you can run into when unmounting disks by command line:

##### _Device or resource is busy_
You may be unable to unmount because the disk is being used something.
```
admin@RT-AC86U:/tmp/home/root# umount -f /tmp/mnt/SANDISK/
umount: can't unmount /tmp/mnt/SANDISK: Device or resource busy
```
This problem is difficult to avoid and the best you can do is try to stop any processes/scripts that may be utilising the disk.
>Quote: "Don't use -f. The device must be cleanly unmounted. If the user cannot unmount the device then they shouldn't proceed. There's no easy way to solve this. It's really up to the user to know what processes are currently using the device and to terminate them. The usual way of identifying such processes is with the fuser command. Unfortunately that isn't part of the normal firmware, although it is in entware." -- ColinTaylor

##### _Ghost devices_
You may find duplicate devices get created.
- umount command does NOT automatically remove the device mount point
- removing mount point must be done manually
- failure to delete the mount point for unmounted devices can result in a confusing list of duplicate "ghost" devices and other problems. 
This problem can be avoided by remembering to manually remove the directories you disk was previously mounted as.
>Quote: "When the router mounts a USB drive it creates a mount point (which is just a directory) with the appropriate name in /tmp/mnt. When you unmount that device using the GUI the router also deletes the mount point. If you unmount the device from the command line you will probably not think to also delete its mount point - that's the problem. If you now physically remove the USB device and then plug it back in the router will look in /tmp/mnt and see that there is already a mount point with the name it wants to use. To avoid mounting the USB drive over the top of (what it thinks is) another device it adds a suffix of (1) to the name it's going to use." -- ColinTaylor

----
### Methods: Automatic or Manual?
There are two (2) methods of formatting disks attached to your router.
1. Automatically with the AMTM - Asus Merlin Terminal Menu script.
2. Manually by following this guide.

Beginners and experts can benefit from using the automatic script to save time and stop human error. Doing the task manually requires reading and understanding what you're doing.

---- 
## Automatic Disk Formatting Script

### amtm - the Asuswrt-Merlin Terminal Menu
 
**amtm** is a shortcut manager for popular scripts for wireless routers running on Asuswrt-Merlin firmware. It is intended to be a helper script, a convenient shortcut manager to install and manage scripts on your router.

The **fd** format disk feature was [beta tested](https://www.snbforums.com/threads/beta-amtm-v1-6_beta-now-with-disk-formatting-automated.54490/page-2#post-459430) and quickly [introduced in Update v1.6](https://www.snbforums.com/threads/amtm-the-snbforum-asuswrt-merlin-terminal-menu.42415/page-20#post-460843).

Visit the [Developer's website](https://diversion.ch/amtm.html) and the [Forum thread](https://www.snbforums.com/threads/amtm-the-snbforum-asuswrt-merlin-terminal-menu.42415/) for information.

---- 
## Manual Step-by-Step Instructions
### 1. Preparations
- The process erases all data and partitions on the device.
- Backup valuable data.
- Format disk with FAT32 filesystem before beginning to ensure it is clean and detectable by ASUS-WRT firmware.
- To be on the safe side, remove all other attached USB devices from router before continuing.

Consider checking the [ASUS Plug-n-Share Disks Compatibility List](https://event.asus.com/2009/networks/disksupport/)

----
### 2. Attach USB disk

Plug your USB storage device into either the USB 2.0 or USB 3.0 port.
- USB 3.0 High Speed port: best for large capacity disks typically used for sharing large media files and video streaming.
- USB 2.0 Low Speed port: suitable for small capacity flash-drives typically used only for router scripts like Diversion, Skynet, Stubby etc.

Tip: Insufficient shielding on USB ports for some router models may cause WiFi interference on the 2.4ghz band. You may choose to enable the Reduce USB Interference option in the router Web UI.

----
### 3. Connect to router via ssh

Login to the router via ssh using your preferred client.

To enable SSH (LAN ONLY) go to [Administration/System](http://router.asus.com/Advanced_System_Content.asp)

***Do not enable SSH for WAN or you will get hacked.***

----
### 4. View disk information

We must begin by checking the information for the attached disk. The information we get here will be used for 

Things to note before continuing:
- **df** command will NOT show unmounted disks.
- **fdisk -l** command will show both mounted and unmounted disks.

**fdisk -- DOS partition maintenance program**
Show the partition table for every disk, including unmounted disks. Include **-l** option to list:
`fdisk -l`

My example below shows **fdisk** output:
```
admin@RT-AC86U:/# fdisk -l

Disk /dev/sda: 15.3 GB, 15376318464 bytes
255 heads, 63 sectors/track, 1869 cylinders
Units = cylinders of 16065 * 512 = 8225280 bytes

Device Boot      Start         End      Blocks  Id System
/dev/sda1               1        1870    15014912   b Win95 FAT32
```
From my output we can see:
1. My disk is **/dev/sda**
2. It contains 1 partition, mounted as **/dev/sda1**
3. Total disk storage capacity is 15.3GB
4. Disk currently uses block size of 512 kb (the default)
5. Disk considers 1 cylinder to be 16065 blocks of 512kb
6. Therefore 1 cylinder equals 8225280 bytes

**df -- display free disk space**
Show human-readable filesystem usage statistics for all _mounted_ disks:
`df -h`

My example below shows **df** output:
```
admin@RT-AC86U:/# df -h
Filesystem                Size      Used Available Use% Mounted on
ubi:rootfs_ubifs         77.2M     63.9M     13.2M  83% /
mtd:bootfs                4.4M      3.3M      1.1M  75% /bootfs
mtd:data                  8.0M    576.0K      7.4M   7% /data
/dev/mtdblock8           48.0M      1.6M     46.4M   3% /jffs
/dev/sda1                14.3G      2.3M     14.3G   0% /tmp/mnt/SANDISK
```
From my output we can see:
1. **/dev/sda1** is mounted at directory **/tmp/mnt/SANDISK**
2. **sda1** currently uses the disk label **SANDISK**

**TAKE NOTE OF YOUR OWN DEVICE DETAILS.**

----
### 5. Unmount

Before we can begin zero'ing the disk, creating new partition tables and formatting we MUST properly unmount the device AND remove it's mount point directory.

 **mount** command will list mounted devices:
`mount`

My example below shows **mount** output:
```
admin@RT-AC86U:/# mount

ubi:rootfs_ubifs on / type ubifs (ro,relatime)
devtmpfs on /dev type devtmpfs (rw,relatime,mode=0755)
proc on /proc type proc (rw,relatime)
tmpfs on /var type tmpfs (rw,noexec,relatime,size=420k)
sysfs on /sys type sysfs (rw,relatime)
debugfs on /sys/kernel/debug type debugfs (rw,relatime)
mtd:bootfs on /bootfs type jffs2 (ro,relatime)
devpts on /dev/pts type devpts (rw,relatime,mode=600)
mtd:data on /data type jffs2 (rw,relatime)
tmpfs on /tmp type tmpfs (rw,relatime)
/dev/mtdblock8 on /jffs type jffs2 (rw,noatime)
/dev/sda1 on /tmp/mnt/SANDISK type ext4 (rw,nodev,relatime)
```
From my output we can see:
1. **/dev/sda1** is mounted at directory **/tmp/mnt/SANDISK**
2. **sda1** currently uses the disk label **SANDISK**
3. **sda1** is formatted with type **ex4** filesystem

Method 1: Use the router web GUI (suggested)
If possible you should try unmount disks from the router web GUI rather instead of umount at the command line interface. It may also be helpful to stop any processes/scripts that may be utilising the disk before attempting to unmount it.

Method 2: Use umount at command line
This method is less reliable and you can [read about it here](#limitations--important-notes). If you encounter problems consider unmounting using the router web UI.

**umount** command unmounts filesystems.

My example device **SANDISK** can unmount with this command:
`umount /tmp/mnt/SANDISK`

If the disk was unmounted properly you should no longer see it listed:
`mount`

Remember there are different ways to check mount status:
- **df** command will NOT show unmounted disks.
- **fdisk -l** command will show both mounted and unmounted disks.

**umount** does **NOT** remove obsolete mount-points for unmounted devices, so we should do it manually.

My example output below lists the contents of the **/tmp/mnt** directory:
```
admin@RT-AC86U:/# ls -l tmp/mnt
drwxrwxrwx    5 admin root          4096 Jan 01 01:00 SANDISK
```
From my output we see my example device has an obsolete mount-point.

The obsolete mount point is removed with this command:
`rmdir /tmp/mnt/SANDISK`

If it was successfully removed then it won't be listed anymore in /tmp/mnt directory:
`ls -l /tmp/mnt`

----
### 6. Zero disk

**This step is OPTIONAL and can cause data loss if done incorrectly.**

Before we begin creating a new partition table it is prudent to properly erase the old partition table. We can achieve this best by overwriting the beginning of the disk with 0's using a utility called **dd** data duplicator.

dd is affectionately nicknamed **DISK DESTROYER** because people can accidentally erase the wrong disk and lose their valuable data. It happens when mistyping the command simply by reversing the source and target disk.

To be on the safe side, remove all other attached USB devices from router before continuing.

**WARNING: RISK OF ACCIDENTAL DATA LOSS IF TYPED INCORRECTLY**.

My example disk **/dev/sda** can be zero'd with this command: `dd if=/dev/zero of=/dev/sda count=16065 bs=512 && sync`

You see we included two (2) options in the command: **bs** is block size in bytes and **count** is the number of blocks to write. Using the default block size of **512** bytes, multiplied by a _block count_ of **16065** we can overwrite the first 8225280 bytes of the disk with zero's. This will completely erase the old partition table and the beginning of the new partition.

Sources:
- [snbforums post-447604](https://www.snbforums.com/threads/diversion-the-router-ad-blocker.48538/page-71#post-447604)
- [snbforums post-459430](https://www.snbforums.com/threads/beta-amtm-v1-6\_beta-now-with-disk-formatting-automated.54490/page-2#post-459430)
- [superuser question-687147](https://superuser.com/questions/687147/how-to-format-a-usb-stick-flash-drive-with-fat32-for-use-on-linux-and-windows/687189#687189).

----
### 7. Repartition disk

**fdisk** command allows managing disk partitions. We must use it to create a new empty MDOS MBR partition table. **Not compatible with disks over 2TB.**

My example device **sda** can be modified with this command:
`fdisk /dev/sda`

From interactive menu, do these commands in order:

**m** (or **help**) to see options:
`m`

**p** to see all existing partitions on disk (take note):
`p`

**o** to erase and create new disk partition table of type mdos MBR Master Boot Record:
`o`

**p** to see all partitions on disk (now we should see none):
`p`

**n** to begin making new partition:
`n`

...then **p** to make it a primary partition:
`p`

...then **1** (one) for its partition number:
`1`

...then accept default values for first and last cylinder

**p** to see all partitions on disk (now we should see 1 created of type ID 83 Linux):
`p`

**w** to write changes to disk:
`w`

----
### 8. Format and adjust filesystem features

**mke2fs** command is to create an ext2, ext3, ext4 filesystem, usually in a disk partition, where device is the special file corresponding to the device (e.g /dev/sdXX ).

A few command options:
`-t` : set file system type
`-L` : set a new volume label
`-O` : specify a feature to use

For the commands below my example device is **sda1**. Your device may differ.

To format as ext2 filesystem:
`mke2fs -t ext2 /dev/sda1`
or use the alias ASUS built-in:
`mkfs.ext2 /dev/sda1`

To format as ext3 filesystem:
`mke2fs -t ext3 /dev/sda1`
or use the alias:
`mkfs.ext3 /dev/sda1`

To format as ext4 with journaling off:
`mke2fs -t ext4 -O ^has_journal /dev/sda1`

To format as ext4 with journaling on:
`mke2fs -t ext4 -O has_journal /dev/sda1`

**tune2fs** command allows to adjust various filesystem parameters on ext2/ext3/ext4 filesystems.

To add or change disk label:
`tune2fs -L "yourDiskLabel" /dev/sda1`

To disable journaling on ext4:
`tune2fs -O ^has_journal /dev/sda1`

To enable journaling on ext4 omit the ^:
`tune2fs -O has_journal /dev/sda1`

To disable 64bit filesystem compatibility:
`tune2fs -O ^64bit /dev/sda1`

To enable 64bit filesystem compatibility omit the ^:
`tune2fs -O 64bit /dev/sda1`

----
### 9. Reboot and mount disk

Rebooting the router is highly recommended to remount a USB device. You may encounter errors with ASUS-WRT if you try mount manually instead of rebooting.

Reboot ASUS router with this command: 
`/sbin/reboot`

If a reboot isn't possible then try manually mount the device. My example device sda1 with label SANDISK can be mounted with this command:
`mount /dev/sda1 /tmp/mnt/SANDISK`

----
## Thanks

The first version of this guide was [posted to the SNB Forums](https://www.snbforums.com/threads/ext4-disk-formatting-options-on-the-router.48302/page-2#post-455723) and led to the development of the [disk formatting AMTM script](https://www.snbforums.com/threads/amtm-the-snbforum-asuswrt-merlin-terminal-menu.42415/). The guide is now maintained here on the Merlin Wiki.

Special thanks to thelonelycoder and ColinTaylor