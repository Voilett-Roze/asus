## Disclaimer
The solutions and opinions expressed in this guide are those of the author (latenitetech), unless or until someone else edits this particular wiki article.  This is not blessed by RMerlin or anyone else of authority, but provided by a relative noobie primarily aimed at other noobies to help them make the most of this amazing compilation of software.  Use at your own risk, or move on if this is too daunting for you.

## Introduction
This article attempts to explain several common issues encountered when using the built-in media server functionality in AsusWRT (aka minidlna).  As many noobies to an AsusWRT-based router, once I had the router set up and working as a router, I then started to explore what else it could do for me.  The media server functionality calls out as what should be an easy to setup and use capability.  Unfortunately, the "out of the box" implementation falls short, and can be a frustrating experience.  It is my hope that understanding the inherent issues will help you fine-tune your personal configuration so that minidlna will operate smoothly.  At the end of this article, I offer up a comprehensive solution for minidlna installation that addresses the issues covered, yet maintains the Web GUI interface and functionality provided by AsusWRT.

First, let me state I've become a fan of minidlna.  When configured properly, it is lightweight, stable and reliable.  Running on the router, its "always on" availability is a great alternative to a PC-based media server consuming 200+ watts, most of the time just idling wasting electricity.  Minidlna is best suited as a music and/or image (photo) server.  While is can also serve up video files, they must be encoded "ready to play" for your playback device, as minidlna doesn't support transcoding.  PC-based media servers (Plex is my fav), are best for a video library where on-the-fly transcoding is often required.  So in my household, I use minidlna on the router to serve up music (always on, always available), and then wake up my PC-based Plex media server to serve up my video library (which otherwise is sleeping to save electricity).

This article focuses on the minidlna built into the AsusWRT-Merlin firmware.  Most of this information also applies to an externally installed instance of minidlna (such as from entware), but some of the defaults may differ.  Frequently you'll read where other users advocate installing an external minidlna thinking the built-in version is inherently flawed.  In reality, the built-in version is just the standard Linux distro, and as of this writing, is actually more current than the version available through entware.  There's no need to install minidlna through entware; just use the built-in one.  Even if you prefer to bypass the Web GUI, and manually configure/start it, you can still run the internal version (located in /usr/sbin).

## The "Out-of-the-Box" Configuration
For those that want the simplest solution, and that doesn't require any use of AsusWRT-Merlin customization, there is only one config that works:  your media library must be fairly small and contained in a single storage volume (partition), on a single USB storage device (flash drive or HDD), and that storage volume must be formatted as a Linux filesystem (i.e. EXT2, EXT3 or EXT4).

A large media library, or one that spans multiple storage devices or volumes, or uses any other file system (e.g. Windows FAT, FAT32 or NTFS) requires additional _under the covers_ configuration utilizing AsusWRT-Merlin's custom scripts to run reliably.  This is not difficult to do, but it's not all that obvious _what to do_ (especially for a noobie), hence the motivation for this article.

## The Three Most Common Issues
### 1)  The minidlna database must reside on a Linux (EXT*) filesystem.

Minidlna utilizes what is loosely referred to as its "database" to build the various menus that are displayed on client media players.  That database is built on initial startup by indexing all files in your media collection.  The database actually consists of two components, an SQL DB (a single file named 'files.db'), and a thumbnail image cache, implemented as a directory tree under "art_cache".  Both of those components, by default, reside in a hidden directory '.minidlna' on the first media storage volume.   While the 'files.db' file isn't particular about the filesystem type (it's just a simple file), the art_cache structure uses file links that are specific to a Linux file system.   It's that latter component the prevents minidlna from successfully creating it's "database" on a non-Linux filesystem.

A typical noobie might tend to take an off-the-shelf USB Flash Drive (nearly always formatted FAT32), throw some media on files on it, plug it into one of the router's USB ports and proceed to enable the media server through the Web GUI.  Minidlna starts up, sees its a new installation and proceeds to build the "database" on that FAT32 filesystem.  It then fails on creation of the art_cache, writing an error message to 'minidlna.log' (also by default located in the "database" directory) for every file indexed.  If you're lucky, it will actually complete the indexing, and files.db is OK, but the art_cache is unusable, leading to downstream client failure.  Your first reaction may be "this is a POS" and abandon it, because you don't understand what's gone wrong with such a simple config.

Note that it's fine for the actual media files (audio, video, image) to reside on a storage device with a Windows filesystem (FAT, FAT32, NTFS), so long as the "database" is located on a Linux filesystem.  In fact, if your desktop environment is Windows, one can make a case that it makes sense to keep your potentially large media collection on Windows-compatible USB storage devices so they can be readily moved to your PC for creation/update.
It's easy to tell minidlna to override the default database location and explicitly place it on a Linux filesystem (it's just one of the entries in the minidlna config file, _/etc/minidlna,conf_), but it requires use of the AsusWRT-Merlin custom scripting capability.  You can see how it's done in my recommended solution at the end of this article.   See the next section on swap if you need help setting up that initial required Linux file system.

One other note here about minidlna.log.  Assuming you get past the issue described above, and eliminate all the errors from art_cache creation, the minidlna.log file can provide excellent information about other legitimate issues with your media library.  In my case, it properly flagged issues with about a half dozen media files that I was unaware of.  Once fixed, my indexing (database creation) now runs clean. 

### 2)  Even a moderately-sized media collection needs swap.

While minidlna is building its database, it can consume considerable memory, especially if you have a large media collection.  When it exhausts available RAM, quite understandably, it fails.  I can't give you a specific size limitation, as it varies widely based on what else is loaded on your router, and what is going on at the same time.  A config on the ragged edge may work one day and fail the next.  The specific errors telling you about this problem is seeing _malloc_  or _out of memory_ errors in your system log.  If you see even one, it's likely telling you it's time to install a swap file.

For those unfamiliar with swap, it's simply a technique to logically expand available memory by allowing the system to temporarily write out inactive memory segments to an external storage device, thereby freeing up memory to be used by active processes.  It does this on-the-fly transparently to the running applications.  Nearly all computers are configured to do this (both desktop and servers).  Windows PCs, for example, are auto-configured for swap by default.  In the router world, however, because there is no external storage equipped out-of-the-box, swap must be manually configured.

The easiest way to install swap is via entware, an add-on software package that greatly expands the capabilities of AsusWRT-Merlin.  AsusWRT-Merlin provides a couple of strategically-placed hooks to integrate entware's features into the base firmware.  It really is an elegant design.  If you're planning to do anything beyond the very basic with your router, you'll want entware, as it opens up a whole new world of possibilities.  

Fortunately, it's pretty easy to do;  the most difficult part for a noobie is just setting up that first Linux filesystem if you're new to the Linux world.  You'll need a Linux filesystem for entware, swap, and as I said in part (1) above, for minidlna.

There are many how-to guides out there for creating your first Linux volume; here's one  I followed:   
http://www.algissalys.com/how-to/format-and-partition-usb-asuswrt-routers

Once you have your EXT* file system available, follow this guide to install entware and the swap file:    
https://www.hqt.ro/how-to-install-new-generation-entware/

Note:  While the AsusWRT-Merlin firmware already contains an entware set-up script as described in:   
https://github.com/RMerl/asuswrt-merlin/wiki/Entware   
It doesn't include the swapfile creation step.  You can still use that, but then you'll need to manually set up the swap file using something like this:   
https://mydevtutorials.wordpress.com/2014/01/10/how-to-activate-swap-on-asus-rt-ac68u-router/

Once you have entware installed, and swap enabled, hopefully those pesky _malloc_ errors will be a thing of the past, and your large media database will complete indexing without error.

### 3)  AsusWRT-Merlin restarts minidlna (with a new .conf file) on every USB storage volume mount.

If you're running minidlna involving more than one storage volume, this is the gotcha that explains why minidlna always recreates it's database on every router reboot.  Minidlna is not designed to work that way; it's just a fallout of the way it's implemented in the router firmware.  A properly functioning installation should build the database once, update on the fly, and survive any number of reboots without rebuilding.

Here's what's happening that's causing this issue:

By design, every time a USB storage device is mounted, the firmware stops minidlna (if it was previously running), recreates the minidlna.conf file (described earlier), then restarts minidlna (now with its  new .conf file).  If you've got minidlna enabled, take a look at that file (_/etc/minidlna.conf_) to help understand what I'm saying here.

That _.conf_ file has numerous lines of interest, but the two that cause the issue are these:
* A line that defines where the database is located:  _db\_dir=_
* And one or more lines that define the media directories:  _media\_dir=_

The _media\_dir_ entries come from the Web GUI, so those are pretty obvious when you look at them.  But the firmware tries to be a little smart and only include lines for media directories that are currently available at the time of this particular restart (i.e.  the volumes that have been mounted so far).

The _db\_dir_ entry is "calculated" on the fly by the firmware.  It is the path to '.minidlna' under the first available _media\_dir_.

So during a boot sequence, minidlna will be restarted multiple times with a varying .conf file, either with the _db\_dir_ directory changing, or the available _media\_dir's_ changing, or both.  Regardless, those changes cause minidlna to abandon the previously generated database, and start from scratch on every reboot.  It will eventually settle down, rebuild the database one more time and you're OK until the next router reboot, when the cycle starts all over again.

What's really needed is for ALL required storage volumes to be mounted BEFORE minidlna is started.  Then it all works correctly.  And if you think about it, that's pretty much a given on a traditional computing system, be it Linux, Windows, or whatever.  Minidlna installed on a traditional Linux box doesn't see this issue.  It's only when plopped into an embedded system like a router that it exhibits this undesirable behavior.

So a common solution to this last problem is to simply insert a delay to allow all storage volumes to mount, and then kick off minidlna startup.  Unfortunately, you can't easily do that and still use the built-in Web GUI functionality for media server control and configuration.  So for that reason, many guides say to abandon the Web GUI and configure and control minidlna outboard (using the custom scripting capability of AsusWRT-Merlin).  And that does indeed work for the most part.

But I'm not going to show anything about that method.  I've come up with an alternate scheme that still fully utilizes the Web GUI to configure and control minidlna, but guarantees that minidlna is started only after all available storage volumes are indeed mounted and available.  It doesn't require a custom firmware build; it's all implemented with just two text files using AsusWRT-Merlin's custom scripting capability.  The next section covers that in detail.


## A Minidlna Configuration Solution for AsusWRT-Merlin routers
Follow this guide for a successful and fulfilling minidlna experience!  This solution addresses all the pitfalls described above, and maintains full use of the stock Web GUI to configure and control minidlna.

### Theory of Operation (aka "How it Works")
This solution exploits the fact that on every USB storage volume mount:
* Minidlna process is killed (if found running)
* The system creates a new minidlna.conf under /etc
* AsusWRT-Merlin calls the custom script _/jffs/scripts/minidlna.postconf_
* Upon return from minidlna.postconf, minidlna is (re)started looking for _/etc/minidlna.conf_
* If minidlna doesn't find _/etc/minidlna.conf_ , it just quietly exits.

The  _/jffs/scripts/minidlna.postconf_ script simply denies minidlna a .conf file until all required storage volumes are available, then it installs a custom minidlna.conf telling it exactly how we want it to run.  

The _/jffs/scripts/minidlna.postconf_ script defines the required storage volumes as the database directory, and all media directories set by the Web GUI (stored in nvram).

Once all required storage volumes are confirmed to be available, the script creates a custom _minidlna.conf_ file by combining parts of the system-generated _.conf_ with user-configurable elements from _minidlna.conf.base_ stored in the minidlna database directory.

Minidlna finally finds a valid _.conf_ file under _/etc_, starts up and happily finds the database still intact from the last run.  All is good.

The Web GUI is fully functional to display the Media Server Status, change media directories, start/stop the server, etc.

### Step by Step
1)  Prerequisites:   
* An available Linux filesystem (EXT2, EXT3 or EXT4), with enough space to hold your minidlna database and a few config files.  (100MB should be plenty, perhaps double or triple that for a huge media collection.)   
* You have _sh_ (command line) access to the router via _telnet_ or _ssh_.   
* If you've not installed entware, and want to proceed anyway, you still need to "Enable support for custom configs."  Follow these guides to learn about this:   
https://github.com/RMerl/asuswrt-merlin/wiki/Custom-config-files   
https://github.com/RMerl/asuswrt-merlin/wiki/User-scripts

2)  Recommended Prereq:  Install entware and swapfile (as described earlier in this article)

3)  Define/create a directory in the Linux filesystem to hold your minidlna database.  I've chosen to use _/opt/app/minidlna_, so if you see that string referenced below, substitute the path to your minidlna database directory if you're using something else.   
[Note _/opt_ is the entware directory, so I just created another directory within the entware structure I call "app" to hold application-specific data, then created another subdirectory within it for minidlna specific files.   This is purely a matter of personal preference, so set it up as you like.]

4)  Copy the contents of the below code block to a new (UNIX-encoded) text file named _minidlna.conf.base_ located under _/opt/app/minidlna_ (your database directory).  Edit the 'db_dir' line to match your minidlna database directory if needed.
    
    # base custom minidlna config file (minidlna.conf.base)
    #
    # friendly_name, media_dir and other system fields are extracted by postconf
    # script from system-generated minidlna.conf and appended to this file, then
    # postconf script replaces the system-generated file with the newly created one
    #
    db_dir=/opt/app/minidlna
    enable_tivo=no
    strict_dlna=no
    inotify=yes
    notify_interval=600
    album_art_names=Cover.jpg/cover.jpg/Thumb.jpg/thumb.jpg
    #root_container=B
    
Note:  for all the new files created here, be sure to follow the script creation guidelines near the bottom of   
https://github.com/RMerl/asuswrt-merlin/wiki/User-scripts   
> _… you must save files with a UNIX encoding.  Note that Windows' Notepad cannot save with a UNIX encoding - get Notepad++ instead.  You can also directly edit them on the router through ssh or telnet, by using vi or nano, both included in the firmware.  These two will create files already encoded in the proper format._

For Windows users, one quick option is to run nano via telnet or ssh, then just copy/paste the contents of the desired file into the nano window, and finally save to your minidlna database directory.

5)  Copy the contents of the below code block to a new (UNIX-encoded) file named _minidlna.postconf_ located under _/jffs/scripts_.
Edit the 'MDLNA_DIR=' line to match your minidlna database directory if needed.
    
    #!/bin/sh
    
    # minidlna.postconf script (to be placed in /jffs/scripts)
    # Checks for existence of required files/directories before allowing minidlna to start normally.
    # If anything is missing (e.g. a required volume hasn't mounted yet), this script ensures there won't
    # be a .conf file (under /etc) available when the system auto-starts minidlna, so it will just abort
    # rather than try to start without all required resources available.
    #
    # Once all files/directories are available, a custom conf file is created by combining the base custom conf file
    # with key fields extracted from the system-generated conf file.  The custom conf file is then installed
    # allowing minidlna to start.  The key fields extracted are those which are set by the router's Web GUI,
    # thereby retaining full Web GUI functionality transparent to the end-user.
    
    # user directory for minidlna related files (must reside on a Linux storage volume)
    MDLNA_DIR=/opt/app/minidlna
    
    CONF=minidlna.conf
    BASECONF=$MDLNA_DIR/$CONF.base
    SYSCONF=/etc/$CONF.system
    MRGCONF=$MDLNA_DIR/.$CONF.merged
    
    # rename the system-generated conf file; this is what causes minidlna startup to abort if all required
    # resources aren't available and we haven't supplied a custom .conf file in its place
    mv /etc/$CONF $SYSCONF
    
    # Look for availability of required files/directories: BASECONF check ensures entware volume is mounted,
    # as well as the base conf file is available, as it's critical in this scheme.
    # The nvram part verifies all media_dirs are available (it returns all the media directories entered thru
    # the web GUI), and it's ultimately what gets loaded in the 'final' system-generated .conf file we use
    # to build the final custom .conf file.
    for n in $BASECONF `nvram get dms_dir_x | tr \< " "`
    do
    	if [ ! -e "$n" ] ; then
    		logger "$0:" "WARNING: $n not found, forcing abort of minidlna startup."
    		exit 1
    	fi
    done
    
    # should be good to go - create custom minidlna.conf file
    echo "# Custom generated $CONF file by $0.  DO NOT EDIT THIS FILE." > $MRGCONF
    grep -v \# $BASECONF >> $MRGCONF
    grep "network_interface=" $SYSCONF >> $MRGCONF
    grep "port=" $SYSCONF >> $MRGCONF
    grep "presentation_url=" $SYSCONF >> $MRGCONF
    grep "friendly_name=" $SYSCONF >> $MRGCONF
    grep "media_dir=" $SYSCONF >> $MRGCONF
    grep "serial=" $SYSCONF >> $MRGCONF
    grep "model_number=" $SYSCONF >> $MRGCONF
    
    logger "$0:" "Installing custom $CONF in /etc"
    cp $MRGCONF /etc/$CONF
    
    exit $?
    
6)  Make newly created _minidlna.postconf_ file executable:   
> chmod a+rx /jffs/scripts/minidlna.postconf

7)  Open the Router's Web GUI, and go to the minidlna config area (USB Application, Media Services and Servers, 'Media Server' tab, 'Media Server' section)   
* Click the button next to 'Enable UPnP Media Server' (if not already enabled)   
* Set the Media Server Name as desired (that is what is presented to clients)   
* Select 'Manual Media Server Path' (this is required for the _postconf_ script to know which storage volumes are required before it allows minidlna to start)   
* Add Media Server Directories in the block below   
* When all done, hit 'Apply'

This should (re)start the minidlna server and all the magic from the above scripts should happen.  If for some reason it doesn't, I suggest troubleshooting before trying it through a reboot.  You can review the .conf generated by looking at either _/etc/minidlna.conf_ OR _\<your minidlna database directory\>/.minidlna.conf.merge_ (they should be identical).  A corrupt _*.postconf_ script can wreak havoc (and be dangerous) with a reboot.  If you can't find the problem, please disable minidlna through the Web GUI and rename or remove minidlna.postconf before rebooting the router.

If all looks good, go for a reboot and see (in the system log) that minidlna is restarted several times (once for each storage volume mount), but ultimately comes up on the last mount, finds your previous database (stored on the Linux volume), and comes up clean without rebuilding.  Look at _\<your minidlna database directory\>/minidlna.log_ for confirmation all is good.

8)  The file named _minidlna.conf.base_ you created under your minidlna database directory has several options for further customizing your media server.  Since the minidlna server provided in the firmware is a standard Linux distro, you can find man pages describing the available options by just googling for it (e.g 'man minidlna') - you'll get lots of hits.

A couple of options that I like to change are  _root_container=B_ (currently commented out in the version I provided above), or extend the _album_art_names=_ to include all the variations you have in your media library.  Check out the man page for what all the options do and if they're appropriate for your use.  But be careful not to duplicate the options that are extracted from the system-generated _.conf_ file (which normally shouldn't be changed).

9)  Enjoy!

Hopefully you found this article helpful.  
Questions/comments?  Look for me as 'latenitetech' on http://www.snbforums.com/forums/asuswrt-merlin.42/  
Glad to help if I can.

-Mark