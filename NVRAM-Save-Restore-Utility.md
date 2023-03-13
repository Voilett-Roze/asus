## Description #

### This script is now deprecated. `amtm` will refuse installation of nsrum if the firmware is newer than 384.19.

There are times when it is necessary to perform a factory reset, such as when loading firmware where the wireless drivers have been updated.  The Save/Restore configuration options in the firmware **cannot** be used to restore your user settings in this case since there is driver specific information also included in the save configuration.  Therefore the user has been required to reconfigure the personalized settings through the GUI by hand after the update.  Additionally, if incorporating a new router to replace an existing router, even of the same model, the firmware Save/Restore options cannot be used and a fresh setup would be required.

A simple script/utility is available that reads from a configuration (ini) file the nvram variables associated with parts of the user configuration (excluding those unique to the router) and creates a restore script for those variables. This replaces the process of re-entering all the customized settings through the web GUI. 

A 'Migration Mode' can be used when transferring settings between routers. This mode excludes certain variables (such as transmit power), that should evaluated based on the new hardware.

## Requirements ##
A USB drive formatted as ext2, ext3, ext4 with an available partition for the NVRAM Save/Restore Utility installation.

## Installing the Utility ##
The latest version of the utility is hosted on [GitHub](https://github.com/Xentrk/nvram-save-restore-utility) and can be installed using [amtm](https://diversion.ch/amtm.html). amtm also has a utility to format a USB.

## Support ## 
[User NVRAM Save/Restore Utility Support Thread](https://www.snbforums.com/threads/release-nvram-save-restore-utility.61722/)