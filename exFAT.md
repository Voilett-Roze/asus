# Background
exFAT is a filesystem suitable for SSDs and Flash Drives. ExFAT was licensed by Microsoft, so there wasn’t a legal kernel module for many years. However, there is a user space FUSE module for accessing exFAT file systems available in entware.

## setup
1. Install exFAT-fuse from entware via opkg
2. Run fdisk -l to find the drive name with the exFAT fs
3. Create a mount point with mkdir -p /mnt/media
4. Mount the drive with mount.exfat-fuse /dev/<drive name> /mnt/media
5. Optionally, setup this process with a /jffs/scripts/service-start script for step 4