## Disclaimer  
The solution and opinions expressed in this guide are those of the author (latenitetech), unless or until someone else edits this particular wiki article.  This is not blessed by RMerlin or anyone else of authority, but provided by a relative noobie primarily aimed at other noobies to help them make the most of this amazing compilation of software.  Use at your own risk, or move on if this is too daunting for you.

## USB Disk Check at Boot for Routers running AsusWRT-Merlin 
This guide shows one way to set up your router to automatically check your USB storage devices on boot, as mentioned in the Wiki guide for User Scripts:  https://github.com/RMerl/asuswrt-merlin.ng/wiki/User-scripts

It assumes you've already enabled user-scripts (_"scripts must be enabled, under Administration -> System on the webui"_).  It does not depend on entware as it only uses functionality already built into the firmware, but some of the tips included here are related to entware and how it interacts with this solution.

This guide covers the case where you have a mix of file system types, typically at least one Linux filesystem (e.g. EXT2) for entware installation, and then FAT32 or NTFS filesystems that come with most flash drives or external USB storage drives for media files.  It also describes potential pitfalls and solutions, and covers some related topics.

The basic installation is quite trivial; just install the contents of the following code segment into /jffs/scripts/pre-mount.
    
    #!/bin/sh
    # pre-mount script (to be installed in /jffs/scripts)
    # auto-check filesystems during boot
    # first argument is the filesystem to be mounted (e.g. /dev/sda1)
    
    CHKLOG=/var/fsck.log
    
    # determine the type of filesystem being mounted
    FSTYPE=`fdisk -l ${1:0:8} | grep $1 | cut -c55-65`
    
    # determine the appropriate checker for the filesystem
    case "$FSTYPE" in
            Linux )
                    CHKCMD="e2fsck -p" ;;
            Win95* | FAT* )
                    CHKCMD="fatfsck -a" ;;
            HPFS/NTFS )
                    CHKCMD="ntfsck -a" ;;
            * )
                    logger "$0:" "Unknown filesystem type $FSTYPE on $1 - no filesystem check run."
                    exit 1 ;;
    esac
    
    logger "$0:" "Running '$CHKCMD $1' - see output at $CHKLOG"
    echo -e "\nStarting '$CHKCMD $1' at `date`" >> $CHKLOG
    $CHKCMD $1 >> $CHKLOG 2>&1
    
And be sure to follow the script creation guidelines near the bottom of https://github.com/RMerl/asuswrt-merlin.ng/wiki/User-scripts
> _… you must save files with a UNIX encoding. Note that Windows's Notepad cannot save with a UNIX encoding - get Notepad++ instead. You can also directly edit them on the router through SSH or telnet, by using vi or nano, both included in the firmware. These two will create files already encoded in the proper format._

So for Windows users, one quick option is to run nano via ssh or telnet, then just copy/paste the contents of the above script into the nano window, and finally save as /jffs/scripts/pre-mount.
> Make the script executable (_chmod a+rx /jffs/scripts/pre-mount_)   

[I can't tell you how many times I've forgotten that step and am left scratching my head wondering why it doesn’t work!]

The script currently supports all flavors of Linux filesystems (EXT2, EXT3, EXT4 - they're all identified as simply 'Linux'); Windows FAT (FAT16), FAT32, and NTFS.  It currently doesn't handle Apple's hfs although fsck_hfs is provided in the firmware (under /usr/sbin).  [HFS is currently left out solely because I'm not an Apple user and I don't know what string fdisk returns for an HFS filesytem.  Probably just 'HFS', but FAT32 surprised me when it returned 'Win95', so rather than risk getting it wrong, I left it out.  If someone can verify the proper entry, feel free to update this Wiki and remove this comment!]

The output from all runs of *fsck (telling you exactly what they did)  are saved in a log file in RAM in /var/fsck.log (it's actually in /tmp/var, which is linked to /var;  /tmp is a virtual file system in RAM created at boot).  What that all means is that the log file will go away on the next boot, so go look at it if interested before you reboot again.

The script runs the quickest, least intrusive flavor of disk check.  So long as the disk is not marked 'dirty' (it was cleanly dismounted the last time), the disk check just does minimal work and exits, typically under a second.  However, if the 'dirty bit' is set, it will perform an automatic disk check and clean, which may take a while depending on the size of the disk and the number of errors encountered.  My personal philosophy is that if the disk has errors, I'd rather have it take the time to fix it before returning to service, regardless of how long it takes.  Note that the primary function of the router (e.g. serving up internet traffic to connected clients) is not delayed by this as it goes about its business in parallel with this script.

## Gotchas
### 1) Extended Boot Time
Due to the possible variation in boot time created by this script, current add-on functionality may be impacted.  The most likely interaction is the call to entware's rc.unslung to start services installed under entware.  The default installation script for entware installs a simple wait loop in services-start that waits only 30 seconds for the entware volume to become available.  If it times out, it sends this message to the system log _"Could not start Entware."_  The extra time taken to do the disk check(s) may cause that wait loop to fail, and especially so if the fsck actually has to do any disk cleaning work.

Browsing threads under http://www.snbforums.com, you'll find this 'wait loop for entware to start' is an old issue, dating back at least to 2013, and probably before.  And there's a bit of a religious debate about what the right solution is.  The simplest solution, advocated by many long-time users of this environment (and just slightly modifying the default installation method), is just make the wait loop longer.  Changing the _'i=30'_ to _'i=60'_ at the top of the services-start script (the number of seconds to wait) will likely fix a "normal" boot (where the disk check goes by quickly), but other than setting it to many minutes, it's a poor solution for when the disk check has to do serious work.  In that case, you'll likely have to the start the services manually (run _'/opt/etc/init.d/rc.unslung start_' from a telnet or ssh window), or reboot again to get everything started up correctly.  But there is a good alternative (at least I think it's good …), but first read these caveats before pursuing the alternative.

a)  There are certain popular add-on applications that require and/or enforce the original services-start design for entware.  One such application is AB-Solution (https://github.com/RMerl/asuswrt-merlin.ng/wiki/How-to-use-Adblock-using-Pixelserv).  If you use that application (or plan to), skip the alternate solution below as that's not an option.  There are likely other apps with this restriction that I'm not aware of, so you may have to revert back to the 'services-start delay loop' method if you adopt the below alternative and find something doesn't work with it.

b)  Certain routers (the RT-N66U is one such example) can utilize an SD card for the entware volume and when configured this way, has a different boot sequence than routers employing strictly USB storage volumes.  It's been reported that the change recommended below is incompatible with such configurations, so the below method would not apply.

The above caveats aside, I, and others before me, have advocated for eliminating the services-start script altogether and move the entware rc.unslung  call to post-mount, in the same block, after the '/tmp/opt' link is made, and after swap is enabled (if installed).
It looks something like this (substitute the 'sys' in _/tmp/mnt/sys_ for whatever label you chose to use for your "entware volume", and the 'entware' in _$1/entware_ for whatever you named that directory):
    
    #!/bin/sh
    #
    # /jffs/scripts/post-mount
    #
    
    if [ "$1" = "/tmp/mnt/sys" ] ; then
        # set up and run Entware
        ln -nsf $1/entware /tmp/opt
    
        logger "$0:" "Enabling swap on /opt/swap"
        swapon /opt/swap
    
        logger "$0:" "Running rc.unslung to start Entware services ..."
        /opt/etc/init.d/rc.unslung start
    fi
    
This solution completely eliminates the need for the delay loop in services-start (eliminates services-start altogether, as far as entware is concerned), and is effectively equivalent since all the services start wait loop is doing is waiting on that link to '/tmp/opt' to be created in post-mount anyway.  That means you can be assured the entware rc.unslung will always be called, regardless of how long any disk checking takes in the boot sequence.  A boot should always bring the router back to its "normal" state without having to look for that dreaded _"Could not start Entware"_ message in the log.

This solution also has some other beneficial side effects:
* In the services-start wait loop model, there is a race condition between when rc.unslung is kicked off (in services-start) and when swap is enabled (in post-mount).  One would prefer to have swap mounted before services are started, but looking at the log on a 'service-start wait loop' system, you'll see services starting before swap is fully enabled.  That may or may not cause a problem depending on how memory intensive the started service is.

* If you remove and reinstall the USB drive holding entware, the post-mount solution will properly restart entware (rc.unslung is called again upon the remount).  But with the services-start solution, you're forced to either manually re-run rc.unslung, or reboot the router (since services-start only runs once upon boot).

Anyway, as with any religious debate, feel free to go with whichever method feels right to you.  Just offering this alternative.

### 2) FSCK run during boot fails with 'out of memory' error
On a substantial disk clean up run (i.e. your disk is really corrupt when the fsck is run at boot) it may not be possible to get to end-of-game because of insufficient memory.  This can occur on large HDDs and/or with many errors.  Because it's being run at boot, swap (if it exists on your system) likely hasn't loaded yet.

There are a couple of options available to address this:   
a)  Move the drive to another computer (e.g. PC) and run disk check there to clean it up, then reinstall it on the router.   

b)  If you don't have a swap file installed on your router, first do that.  Once you have swap installed and enabled, then manually run a disk check on the disk after the router is up and swap has been activated.  You can use the built-in 'Health Scanner' mentioned below if you prefer to avoid running the required 'umount' and 'fsck' commands from a command line.

## But what about the 'Health Scanner' available in the Web GUI?
It's been pointed out that there already is disk-check functionality built into the AsusWRT firmware, accessible from the Web GUI.  On the Network Map screen, click on the USB drive icon and a panel will open on the right side with a 'Health Scanner' tab.  From there you can initiate a comprehensive disk scan on any of your USB storage devices.  You can also schedule a recurring check from that screen.   A problem with that mechanism is it's part of the original AsusWRT, and doesn't understand the Merlin extensions.  So it doesn't know to first stop entware (meaning the services started by entware and/or a swapfile, if you've enabled that), and then fails rather ungracefully when it can't _umount_ a storage volume.

Here's another simple custom script that makes this functionality work (better, at least).
Install as '/jffs/scripts/unmount' (caveats for installing scripts mentioned above apply):
    
    #!/bin/sh
    #
    # /jffs/scripts/unmount
    #
    # stop entware prior to attempting to unmount the "entware volume"
    #
    
    # determine if this is the entware volume by comparing the '/opt' mountpoint to $1
    OPT=$(dirname $(readlink /tmp/opt))
    if [ "$1" == "$OPT" ] ; then
            # this should be the same code as in 'services-stop', so you could just call services-stop instead
            /opt/etc/init.d/rc.unslung stop
            swapoff /opt/swap
    fi
    
With that addition, the Health  Scanner works better, in that it can now successfully Health Check the entware volume.  It still has some issues though -- I'm pretty sure are in the original AsusWRT code that RMerlin has never gotten around to fixing.  It appears it gets confused as to when to start and stop the built-in Samba and minidlna processes.  On at least one occasion it corrupted my minidlna database, forcing it to recreate it on the next boot.  Not a biggie, but not how it should be.  I haven't taken the time to understand what's really going on here, so I don't trust it to run automatically on a schedule.  But hey, give it a try if you like.  If it works for you, all the power …

Questions/comments?  Look for me as 'latenitetech' on http://www.snbforums.com/forums/asuswrt-merlin.42/  Glad to help if I can.

-Mark
