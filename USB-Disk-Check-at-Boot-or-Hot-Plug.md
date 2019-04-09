# This is work in progress and not yet published!

## Background
This guide is a followup to the original found [here](https://github.com/RMerl/asuswrt-merlin/wiki/USB-Disk-Check-at-Boot). The purpose of this guide is to provide example `pre-mount` scripts that can be used to automatically check and repair the filesystems of USB storage devices during boot or when they are plugged in. This might be desirable because after the router has automatically mounted a filesystem it can sometimes be difficult to unmount it for checking. This is especially true if that filesystem is used by third-party software such as Entware.

This guide attempts to address the following limitations present in the original script:
1. It uses MBR partition IDs to identify the filesystem type. Unfortunately there is no guarantee that the partition ID actually matches the filesystem present in the partition.
2. It is not compatible with GPT disks.
3. It uses a command to check NTFS filesystems that may not be present on certain router models.
4. It makes no attempt to check HFS filesystems.

Since Merlin's 384.11?? and John's V39E1 firmware releases a second parameter (`$2`) is passed to the `pre-mount` user script. This parameter contains the filesystem type as detected by the router. This should be much more reliable than using the partition ID. The following is a list of all the possible types: `NULL`, `mbr`, `swap`, `ext2`, `ext3`, `ext4`, `hfs`, `hfs+j`, `hfs+jx`, `ntfs`, `apple_efi`, `vfat`, `unknown`.

The first parameter is unchanged and is the device name (e.g. `/dev/sda1` or `/dev/sda`). Please note that it is valid to have a device name that doesn't end in a digit (e.g. `/dev/sda`). This typically indicates that the device has no partition table and contains a single filesystem (i.e. a Super Floppy) as commonly seen with USB flash drives.
## Prerequisites
The following firmware versions or higher are required: Merlin's 384.11?? or John's V39E1.

## Considerations
1. Where possible the example scripts perform a "quick" check of the filesystem. This is usually desirable because `pre-mount` is a blocking script and performing a full filesystem check could detrimentally effect boot times.
2. When using the "Disk Scan" utility in the router's webUI bear in mind that after it has completed the device is remounted causing `pre-mount` to run again. This could mean that the device is checked a second time!

## Example scripts

The following can be used as-is or as the basis for your own `pre-mount` script. I won't describe how to create or enable [user scripts](https://github.com/RMerl/asuswrt-merlin/wiki/User-scripts) as that is explained elsewhere in the wiki. I have tried to write these examples in such a way that it is easy to understand how they work. They deliberately _don't_ use "clever" (i.e. non-obvious) coding techniques.
### Simple script
This script only checks ext2, ext3 and ext4 filesystems and sends all output to the router's syslog.

    #!/bin/sh

    # $1=device $2=filesystem type

    TAG="$(basename $0)"

    if [ "$2" = "ext2" ] || [ "$2" = "ext3" ] || [ "$2" = "ext4" ]; then
        CHKCMD="e2fsck -p $1"
        logger -t "$TAG" "Running '$CHKCMD'"
        $CHKCMD 2>&1 | logger -t "$TAG"
    fi

### Advanced script
This script contains code blocks for every possible filesystem type to help document the meaning of some of the less obvious ones. Indeed, I would encourage you to remove sections that you're not interested in (e.g. `""`, `mbr`, `swap`, `hfs`, `apple_efi` and `unknown`) in order to keep _your_ script more manageable. Informational messages are sent to the router's syslog but the actual output from the fsck commands are sent to `/var/log/fsck.log`.

    #!/bin/sh

    # $1=device $2=filesystem type

    TAG="$(basename $0)"
    CHKLOG="/var/log/fsck.log"

    if [ $# -lt 2 ]; then
        logger -t "$TAG" "Missing paramter. Firmware too old?"
        exit 1
    fi

    case "$2" in
        "")
            logger -t "$TAG" "Error reading device $1"
            exit 1 ;;
        mbr)
            logger -t "$TAG" "No partitions found in MBR ($1)"
            exit 1 ;;
        swap)
            logger -t "$TAG" "$1 is a swap partition"
            exit 0 ;;
        ext2|ext3|ext4)
            CHKCMD="e2fsck -p" ;;
        hfs|hfs+j|hfs+jx)
            hfs_mod="$(nvram get usb_hfs_mod)"
            if [ "$hfs_mod" = "open" ]; then
                CHKCMD="fsck.hfsplus -f"
            elif [ "$hfs_mod" = "paragon" ]; then
                CHKCMD="chkhfs -a -f"
            elif [ "$ntfs_mod" = "tuxera" ]; then
                CHKCMD="fsck_hfs -fy"
            else
                exit 1
            fi ;;
        ntfs)
            ntfs_mod="$(nvram get usb_ntfs_mod)"
            if [ "$ntfs_mod" = "paragon" ] || [ -z "$(nvram dump | grep usb_ntfs_mod)" ]; then
                CHKCMD="chkntfs -a -f"
            elif [ "$ntfs_mod" = "tuxera" ]; then
                CHKCMD="ntfsck -a"
            else
                exit 1
            fi ;;
        apple_efi)
            logger -t "$TAG" "$1 is an Apple EFI system partition"
            exit 0 ;;
        vfat)
            CHKCMD="fatfsck -a" ;;
        unknown)
            logger -t "$TAG" "$1 has an unknown filesystem (e.g. exFAT) or no partition table (e.g. blank media)"
            exit 1 ;;
        *)
            logger -t "$TAG" "Unexpected filesystem type $2 for $1"
            exit 1 ;;
    esac

    logger -t "$TAG" "Running '$CHKCMD $1' - see output in $CHKLOG"
    echo -e "\nStarting '$CHKCMD $1' at $(date)" >> $CHKLOG
    $CHKCMD "$1" >> $CHKLOG 2>&1

This advanced script borrows heavily from the router's own built-in disk checker, `/usr/sbin/app_fsck.sh`. In fact you may want to use `pre-mount` as a simple wrapper for that built-in script. However, that script does have some limitations or may use options that you find undesirable.