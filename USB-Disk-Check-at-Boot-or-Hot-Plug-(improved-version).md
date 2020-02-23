## Background
This guide is a followup to the original found [here](https://github.com/RMerl/asuswrt-merlin.ng/wiki/USB-Disk-Check-at-Boot). The purpose of this guide is to provide example `pre-mount` scripts that can be used to automatically check and repair the filesystems of USB storage devices during boot or when they are plugged in. This might be desirable because after the router has automatically mounted a filesystem it can sometimes be difficult to unmount it for checking. This is especially true if that filesystem is used by third-party software such as Entware.

These scripts contain the following improvements over the original script:
1. Compatibility with GPT disks (GPT is required for disks larger than 2TB).
2. Reliable detection of the filesystem type.
3. Disk check commands compatible with both Merlin's and [John's firmware](https://www.snbforums.com/threads/fork-asuswrt-merlin-374-43-lts-releases-v39e1.18914/).
4. Support for HFS filesystems.
5. Detection of swap partitions.

Since Merlin's 384.11 and John's V39E1 firmware releases a second parameter (`$2`) is passed to the `pre-mount` user script. This parameter contains the filesystem type as detected by the router. This is much more reliable than the previous method of using the MBR partition ID. The following is a list of all the possible filesystem types: `NULL`, `mbr`, `swap`, `ext2`, `ext3`, `ext4`, `hfs`, `hfs+j`, `hfs+jx`, `ntfs`, `apple_efi`, `vfat`, `unknown`.

The first parameter (`$1`) is unchanged and is the device name (e.g. `/dev/sda1` or `/dev/sda`). Please note that it is valid to have a device name that doesn't end in a digit (e.g. `/dev/sda`). This typically indicates that the device has no partition table and contains a single filesystem (i.e. a Super Floppy) as commonly seen with USB flash drives.
## Prerequisites
The following firmware versions or higher are **required**: Merlin's 384.11 or John's V39E1.

## Considerations
1. Where possible the example scripts perform a "quick" check of the filesystem. This is usually desirable because `pre-mount` is a blocking script and performing a full filesystem check could detrimentally effect boot times.
2. By default a "full" check of an ext2, ext3 or ext4 filesystem is forced if it has been mounted more than 20 times or more than six months has passed since the last check. This behaviour can be changed with the `tune2fs` command.
3. When using the "Disk Scan" utility in the router's webUI bear in mind that after it has completed the device is remounted causing `pre-mount` to run again. This could mean that the device is checked a second time!

## Example scripts

The following can be used as-is or as the basis for your own `pre-mount` script. I won't describe how to create or enable [user scripts](https://github.com/RMerl/asuswrt-merlin.ng/wiki/User-scripts) as that is explained elsewhere in the wiki. I have tried to write these examples in such a way that it is easy to understand how they work. They deliberately _don't_ use "clever" (i.e. non-obvious) coding techniques.
### Simple script
This script only checks ext2, ext3 and ext4 filesystems and sends **all** output to the router's syslog.

    #!/bin/sh

    # $1=device $2=filesystem type

    if [ "$2" = "ext2" ] || [ "$2" = "ext3" ] || [ "$2" = "ext4" ]; then
        logger -t "pre-mount" "Checking filesystem on $1"
        e2fsck -p "$1" 2>&1 | logger -t "pre-mount"
    fi

### Typical script
This script is a cut down version of the Advanced script and is probably appropriate for most users. It contains only the sections that perform the checks. Types of `""`, `mbr`, `swap`, `apple_efi` and `unknown` are ignored. If you don't use HFS (i.e. Apple) disks you might want to remove that section of the code as well. Informational messages are sent to the router's syslog but the actual output from the fsck commands are sent to `/var/log/fsck.log`.

    #!/bin/sh

    # $1=device $2=filesystem type

    TAG=$(basename "$0")
    CHKLOG=/var/log/fsck.log
    CHKCMD=""

    case "$2" in
        ext2|ext3|ext4)
            CHKCMD="e2fsck -p" ;;
        hfs|hfs+j|hfs+jx)
            if [ -x /usr/sbin/chkhfs ]; then
                CHKCMD="chkhfs -a -f"
            elif [ -x /usr/sbin/fsck_hfs ]; then
                CHKCMD="fsck_hfs -d -ay"
            fi ;;
        ntfs)
            if [ -x /usr/sbin/chkntfs ]; then
                CHKCMD="chkntfs -a -f"
            elif [ -x /usr/sbin/ntfsck ]; then
                CHKCMD="ntfsck -a"
            fi ;;
        vfat)
            CHKCMD="fatfsck -a" ;;
    esac

    if [ -n "$CHKCMD" ]; then
        logger -t "$TAG" "Running '$CHKCMD $1' - see output in $CHKLOG"
        echo -e "\nStarting '$CHKCMD $1' at $(date)" >> $CHKLOG
        $CHKCMD "$1" >> $CHKLOG 2>&1
    fi


### Advanced script
This script contains code blocks for every possible filesystem type to help document the meaning of some of the less obvious ones. I would encourage you to remove sections that you're not interested in in order to keep _your_ script more manageable. Informational messages are sent to the router's syslog but the actual output from the fsck commands are sent to `/var/log/fsck.log`.

    #!/bin/sh

    # $1=device $2=filesystem type

    TAG=$(basename "$0")
    CHKLOG=/var/log/fsck.log
    CHKCMD=""

    if [ $# -lt 2 ]; then
        logger -t "$TAG" "Missing parameter. Firmware too old?"
        exit 1
    fi

    case "$2" in
        "")
            logger -t "$TAG" "Error reading device $1" ;;
        mbr)
            logger -t "$TAG" "No partitions found in MBR ($1)" ;;
        swap)
            logger -t "$TAG" "$1 is a swap partition" ;;
        ext2|ext3|ext4)
            CHKCMD="e2fsck -p" ;;
        hfs|hfs+j|hfs+jx)
            if [ -x /usr/sbin/chkhfs ]; then
                CHKCMD="chkhfs -a -f"
            elif [ -x /usr/sbin/fsck_hfs ]; then
                CHKCMD="fsck_hfs -d -ay"
            fi ;;
        ntfs)
            if [ -x /usr/sbin/chkntfs ]; then
                CHKCMD="chkntfs -a -f"
            elif [ -x /usr/sbin/ntfsck ]; then
                CHKCMD="ntfsck -a"
            fi ;;
        apple_efi)
            logger -t "$TAG" "$1 is an Apple EFI system partition" ;;
        vfat)
            CHKCMD="fatfsck -a" ;;
        unknown)
            logger -t "$TAG" "$1 has an unknown filesystem (e.g. exFAT) or no partition table (e.g. blank media)" ;;
        *)
            logger -t "$TAG" "Unexpected filesystem type $2 for $1" ;;
    esac

    if [ -n "$CHKCMD" ]; then
        logger -t "$TAG" "Running '$CHKCMD $1' - see output in $CHKLOG"
        echo -e "\nStarting '$CHKCMD $1' at $(date)" >> $CHKLOG
        $CHKCMD "$1" >> $CHKLOG 2>&1
    fi

***

The Typical and Advanced scripts borrow heavily from the router's own built-in Disk Scan utility, `/usr/sbin/app_fsck.sh`. In fact you may want to use `pre-mount` as a simple wrapper for that script instead. However, that script does have some limitations and may use options that you find undesirable such as performing a "full" check.