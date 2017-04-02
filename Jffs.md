JFFS is a writeable section of the flash memory (the size will vary between router models, with the newer models having a bit over 60 MB of space available), which will allow you to store small files (such as scripts) inside the router without needing to have a USB disk plugged in.  This space will survive reboot (but it might **NOT** survive firmware flashing, so back it up first before flashing!).  It will also be available fairly early at boot (before USB disks).

Starting with 378.50, this option is enabled by default.  If for some reason you need to erase its content, you can do so from the Administration page, under the System tab.  Formatting the JFFS partition requires a reboot to take effect.  Note that formatting it might possibly require a second reboot afterwards, if it fails to properly mount after that first reboot.

I do not recommend doing frequent writes to this area, as it will prematurely wear out the flash chip.  This is a good place to put files that are written once like scripts or kernel modules, or that rarely get written to.  Do not put files that get constantly written to (such as high activity logfiles) - store these on a USB disk instead.  Replacing a worn out USB flash disk is much cheaper than replacing the whole router if flash sectors get worn out - they have a limited number of write cycles.

# Backing up the JFFS Partition
Backing up the JFFS Partition is an easy thing to do under the firmware its recommended to do if your planning to update your firmware to a newer version or if your changing scripts or configurations, Openvpn keys on your router is located under the menu 

Administration - Restore/Save/Upload Setting

On that menu you can either Backup JFFS partition with every file and config on it or Restore JFFS partition a previous partition incase of a accidental wipe.

its highly recommended if you have a configuration or a script that you like that you store a copy of your jffs partition incase if the router resetting or an accidental wipe.

See also: [Restore Settings / Factory Reset / Migrating Settings](https://github.com/RMerl/asuswrt-merlin/wiki/User-NVRAM-Save-Restore-Utility) for backing/restore up NVRam Settings