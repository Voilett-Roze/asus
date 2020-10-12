**IMPORTANT:** NFS sharing is still a bit unstable.

In addition to SMB and FTP, you can now also share any plugged hard disk through NFS. The NFS Exports interface can be accessed from the USB Applications section, under Servers Center. Click on the NFS Exports tab.

Select the folder you wish to export by clicking on the Path field. Under Access List you can enter IPs/Networks to which you wish to give access.  A few examples:

    192.168.1.0/24 - will give access to the whole local network
    192.168.1.10 192.168.1.11 - will give access to the two IPs (separate with spaces)

Entering nothing will allow anyone to access the export.

Under options you can enter the export options, separated by a comma. For example:

    rw,sync

For more info, search the web for documentation on the format of the /etc/exports file.  The same syntax for the access list and the options is used by the webui.

You can also manually generate an exports file by creating a file named _/jffs/configs/exports.add_ and entering your standard exports there. They will be added to any exports configured on the webui.

Note that by default, only NFSv3 is supported.  You can also enable NFSv2 support from that page, but this is not recommended, unless you are using an old NFS client that doesn't support V3.  NFSv2 has various 
filesystem-level limitations.

**Note**: Only native Linux filesystems like ext3 (etc) may be exported (e.g. NTFS is not supported)