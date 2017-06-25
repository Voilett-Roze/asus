## Disclaimer
The solutions and opinions expressed in this guide are those of the author (latenitetech), unless or until someone else edits this particular wiki article.  This is not blessed by RMerlin or anyone else of authority, but provided by a relative noobie primarily aimed at other noobies to help them make the most of this amazing compilation of software.  Use at your own risk, or move on if this is too daunting for you.

## Introduction
pyTivo is a media server for Tivo devices.  You can learn all about it here:   
http://pytivo.sourceforge.net/wiki/index.php/PyTivo

Running on the router, its "always on" availability is a great alternative to a PC-based media server consuming 200+ watts, most of the time just idling wasting electricity.  pyTivo on the router is best suited as a music and/or image (photo) server. While is can also serve up video files, it's best if they are already encoded "ready to play" on the Tivo device.  While pyTivo does support transcoding (via ffmpeg), the limited processing power of the router makes it rather unsatisfactory.  A PC-based pyTivo installation is best for a video library where on-the-fly transcoding is required. So in my household, I use pyTivo on the router to serve up music (always on, always available), and then wake up my PC-based pyTivo installation to serve up my (much larger) video library (which otherwise is sleeping to save electricity).  Both instances of pyTivo can exist and run in parallel with no issue, so long as the shares have unique names.

On my Asus router, I have both pyTivo and minidlna running, both serving up the identical media libraries (installed on large flashdrives mounted on the router).  They are servers to different clients:  pyTivo to Tivo boxes, minidlna to DLNA players.  I happen to have both in my household.

If you've looked closely at minidlna, you might have noticed it also supports Tivos as well (config option _enable_tivo=yes/no_).  Well it does (and it actually works fairly well), at least on older Tivo models.   Here are the issues with minidlna in this space:
* Over its long lifespan, Tivo has employed two different methods for communication on a LAN.  The original design, called _Tivo Beacon_ was a propriety broadcast mechanism.  It is what minidlna uses when Tivo support is enabled.  It still works on the Premiere and older models.
* Tivo has since migrated to the industry-standard zeroconf mechanism.  zeroconf is also known as _Bonjour_ and _Rendezvous_ in the Apple world, and _Avahi_ in the Linux world.  Tivo requires zeroconf with its current Roamio and Bolt models.  For that reason minidlna is incompatible with current Tivo models.
* pyTivo supports both the _Tivo Beacon_ and _ZeroConf_ communication protocols, so it works on all Tivo models.
* Finally, even if you only have Premiere or older Tivo models, pyTivo offers a more robust user interface and more configuration options than minidlna.

Installing pyTivo is pretty straight-forward.  It is written in python, an interpreted language, so it doesn't require any native compilation.  It just requires a  python interpreter, which thankfully is readily available via Entware.  It's also strongly encouraged to install ffmpg, which is also available via Entware.  So basically, install the software components, make some fairly simple configuration changes on your router, and you're up and running.

## Zeroconf (this is techie background - you can skip it if you like)
The zeroconf protocol utilizes Multicast DNS (mDNS), specifically using the multicast address space 224.0.0.0.  However, if you look at the default routing table on the router (the _route_ command) , you will see the AsusWRT firmware doesn't provide a default route for this mDNS broadcast traffic (which makes sence as it is unneeded for doing its primary role as a router).  However, out-of-the-box, this causes pyTivo to not work with current-generation Tivo boxes that depend on zeroconf.  (Older Tivos that use _Tivo Beacon_ work fine.)  It took me quite a bit of web searching and experimentation to figure this problem out, but once I found the elusive post that explained what was happening, the fix is trivial:  just add the one-line route to the routing table.  You'll see it below in the 'Step by Step' section.

## Step by Step
1)  Prerequisites:   
* A Linux filesystem in which you've installed entware and a swap file (the latter isn't critical, but python does consume a bit of memory) 
* You have _sh_ (command line) access to the router via _telnet_ or _ssh_.    

2)  Via a telnet or ssh prompt,  install python, ffmpeg. and pgrep through entware:
    
    opkg install python
    opkg install ffmpeg
    opkg install procps-ng
    opkg install procps-ng-pgrep
       
'pgrep' isn't technically required, but it's used in the init.d start-up script.  It's a handy command to have around anyway.

3)  Define/create a directory in the Linux filesystem to hold your pyTivo software and configuration.  I've chosen to use _/opt/app/pyTivo_, so if you see that string referenced below, substitute the path to your pyTivo directory if you're using something else.   
[Note _/opt_ is the entware directory, so I just created another directory within the entware structure I call "app" to hold application-specific data, then created another subdirectory within it for pyTivo specific files.   This is purely a matter of personal preference, so set it up as you like.]

4)  Bring over a copy of pyTivo and install it in the directory defined in the previous step.    
* If you're new to pyTivo, I suggest using this guide to obtain the software:   
http://pytivo.sourceforge.net/wiki/index.php/Current_Releases   
Then follow the instructions on that page under:  _New users are recommended to start here_

* If you've already got pyTivo running somewhere else (e.g. a Windows box), it's actually OK just to copy that folder over.  You'll just need to delete the 'bin' directory as that contains binaries for the target environment (e.g. Windows), which are obviously not compatible with the router.  All the other files in the distro are just text files.

5)  Edit the pyTivo.conf file as recommended in the previous instructions.  Mine is shown below for example.    
\- On the _beacon_ line, update to match your network's subnet (the '192.168.1' part, the 255 at the end you'll want to leave as that's the broadcast identifier)   
\- Update the media shares to match your rig.  The string in the square brackets is the name displayed on the Tivo client.   
    
    [Server]
    beacon = 192.168.1.255
    ffmpeg = /opt/bin/ffmpeg
    
    [Music on Router]
    type = music
    path = /mnt/AUD/Music
    
    [Podcasts on Router]
    type = music
    path = /mnt/AUD/Podcasts
    
    [Video on Router]
    force_alpha = on
    type = video
    path = /mnt/VID/Video
    
6)  Enable zeroconf broadcast to work (as explained above in the section titled "Zeroconf"):
    
    route add -net 224.0.0.0 netmask 224.0.0.0 br0
    
7)  Try to start pyTivo in a telnet window:
    
    cd /opt/app/pyTivo
    python pyTivo.py
    
You should see something like this coming back from pyTivo:
> INFO:pyTivo:Last modified: Mon Nov 14 02:07:58 2016   
> INFO:pyTivo:Python: 2.7.12   
> INFO:pyTivo:System: Linux-2.6.36.4brcmarm-armv7l-with-glibc2.4   
> INFO:pyTivo.beacon:Scanning for TiVos...   
> INFO:pyTivo.beacon:FamRM Bolt   
> INFO:pyTivo.beacon:MBR Premiere   
> INFO:pyTivo.beacon:Announcing shares...   
> INFO:pyTivo.beacon:Registering: Music on Router   
> INFO:pyTivo.beacon:Registering: Podcasts on Router   
> INFO:pyTivo.beacon:Registering: Video on Router   
> INFO:pyTivo:pyTivo is ready.   

Optional: to get to pyTivo's Web GUI as a user-friendly way to edit advanced config options, navigate to http://\<your router's IP>\:9032/   
For example, http://192.168.1.1:9032/

8) Go to your Tivo and see if the shares show up.    
* Music and Photo shares will appear on the 'Music and Photos' menu.
* Video shares will appear at the very bottom of your 'My Shows' list.

Select a share, navigate to a media file and start it playing.  Sit back and enjoy your new capability!

A common problem with recent Tivos (those depending exclusively on zeroconf) is they are slow to respond to new media servers coming onto the network.  You can force a zeroconf refresh by 'bouncing' the network on the Tivo device (e.g. unplug the network cable, wait a few seconds, then plug it in again).  You should then see the pyTivo shares pop up on your Tivo's menu in a couple of seconds.

9)  Once you're happy with your configuration, install the following start script on the router so it will start up automatically.   
\- Update the line _'PYTIVO_DIR='_ to match your config if it's not correct as is.    
\- Install to:  _/opt/etc/init.d/S50pyTivo_   
(Feel free to change the '50' to whatever number you want, it determines the start-up order of services under init.d.  Lower-numbered are started first.)

Create this file (in UNIX-format, as described in  https://github.com/RMerl/asuswrt-merlin/wiki/User-scripts)   
> _… you must save files with a UNIX encoding.  Note that Windows' Notepad cannot save with a UNIX encoding - get Notepad++ instead.  You can also directly edit them on the router through ssh or telnet, by using vi or nano, both included in the firmware.  These two will create files already encoded in the proper format._

For Windows users, one quick option is to run nano via telnet or ssh, then just copy/paste the contents of the desired file into the nano window, and finally save to the desired directory.
    
    #!/bin/sh
    
    # pyTivo init.d script
    # To be installed as /opt/etc/init.d/SXXpyTivo
    # XX in file name reflects start order (e.g. S50pyTivo)
    
    # the path to your pyTivo installation directory       
    PYTIVO_DIR=/opt/app/pyTivo
    
    RETVAL=0
    start() {
    echo -n "Starting pyTivo: "
    pgrep -f pyTivo.py
    RETVAL=$?
    [ $RETVAL -eq 0 ] && echo "pyTivo already running: Exiting" && exit 1
    
    if [ -e $PYTIVO_DIR/NOLOG ]
    then
    	PYTIVO_LOG=/dev/null
    	logger "Entware init.d:" "Starting pyTivo, logging disabled"
    else
    	PYTIVO_LOG=$PYTIVO_DIR/pyTivo.log
    	logger "Entware init.d:" "Starting pyTivo, logging enabled"
    fi
    
    # Add route for broadcast packets to/from router (required for zeroconf to work w/pyTivo)
    route add -net 224.0.0.0 netmask 224.0.0.0 br0
    
    # this call actually starts pyTivo.
    nohup python $PYTIVO_DIR/pyTivo.py > $PYTIVO_LOG 2>&1 &
    RETVAL=$?
    [ $RETVAL -eq 0 ] && echo -n "done"
    echo
    return $RETVAL
    }
    
    stop() {
    echo -n "Stopping pyTivo: "
    PID=`pgrep -f pyTivo.py`
    [ "x$PID" == "x" ] && echo "pyTivo not found running: Exiting" && exit 1
    logger "Entware init.d:" "Stopping pyTivo"
    kill $PID
    RETVAL=$?
    if [ $RETVAL -eq 0 ]
    then
    	# Delete route previously added by 'start' block
    	route del -net 224.0.0.0 netmask 224.0.0.0 br0
    	RETVAL=$?
    fi
    echo
    [ $RETVAL -eq 0 ] && echo -n "done"
    echo
    return $RETVAL
    }
    
    # See how we were called.
    case "$1" in
    start)
    start
    ;;
    stop)
    stop
    ;;
    restart|reload)
    stop
    sleep 1
    start
    RETVAL=$?
    ;;
    check)
    pgrep -fa pyTivo.py
    RETVAL=$?
    ;;
    showlog)
    tail -120 $PYTIVO_DIR/pyTivo.log
    RETVAL=$?
    ;;
    taillog)
    tail -50 -f $PYTIVO_DIR/pyTivo.log
    RETVAL=$?
    ;;
    *)
    echo "Usage: $0 \{start\|stop\|restart\|check\|showlog\|taillog\}"
    exit 1
    ;;
    esac
    exit $RETVAL
    
10)  Make that start-up script executable, then start pyTivo with it (make sure you have exited the manually started instance you were testing with):
    
    chmod a+rx /opt/etc/init.d/S50pyTivo
    /opt/etc/init.d/S50pyTivo start
    
It now should auto-start on router reboot as well.

11)  The start-up script provides an easy way to enable or disable logging from pyTivo.  I suggest leave it by default to create a log file under your pyTivo directory until you're sure it's running OK.  But it can grow quite large as it logs a lot of messages just from normal activity.  Once you're satisfied it's working correctly, you can easily turn off logging by creating a dummy file in the pyTivo directory named _NOLOG_, then restarting pyTivo
    
    touch /opt/app/pyTivo/NOLOG
    /opt/etc/init.d/S50pyTivo restart
    
Re-enable logging by deleting (or renaming) _NOLOG_ and then restart pyTivo via the init.d script.

12)  Enjoy!

***

## Avahi-Daemon (Optional)
pyTivo, as implemented, is a self-contained, stand-alone zeroconf solution.  It does not depend on, or in any way use the built-in avahi-daemon process running by default under AsusWRT-Merlin.  Near as I can tell, the built-in avahi-daemon only exists to support the _iTunes_ and _Time Machine_ optional services (both targeted to Apple users), yet it runs by default on all routers even if you have both of the Apple-oriented services disabled.  There are comments in the pyTivo zeroconf code that says it's compatible with avahi-daemon, and I have no reason to doubt that, but since I don't use those Apple services,  I prefer to kill off the avahi-daemon process both to save system resources, and to ensure I don't get any undesirable interaction between those competing zeroconf stacks.  If you are using either Apple service, then you do indeed need the avahi-daemon process running, but fair warning you may see interaction between pyTivo and those Apple services.  Hopefully not, but I've not had experience one way or another.

I'm not aware of a way to _not_ start the avahi-daemon process by default, as it's built into the firmware.  But I have learned that, like most Linux services, if a service starts up and doesn't find it's required _.conf_ file it will quietly exit.  So the easy fix is just remove the default  avahi-daemon.conf file at boot.  You will see it starting in the log file, but then it immediate exits and you won't see a trace of it running with _ps_ or _pgrep_.  At the time of this writing, I've been running this way for over a month with no observable downside, at least in my rig.


The simplest 'kill off avahi-daemon' solution is to install an empty avahi-daemon.conf file in /jffs/configs:
    
    touch /jffs/configs/avahi-daemon.conf
    
That's it, you're done.  When the router reboots, it will install the empty .conf file on top of the system-generated one.

A slightly more elaborate arrangement is to install an _avahi-daemon.postconf_ script under /jffs/scripts.  That's what I do:   
[All caveats noted above about creating in UNIX-format, and making the script executable apply ...]   
    
    #!/bin/sh
    #
    # /jffs/scripts/avahi-daemon.postconf
    #
    # WE DON'T WANT AVAHI-DAEMON RUNNING.  It serves no purpose in our use, and may in fact interfere with
    # the zeroconf service running as part of pyTivo.  This script's only purpose is to effectively remove
    # the system-generated .conf file so when avahi-daemon attempts to start, it will die a quick death.
    
    # save the system-generated conf file (in case we want to review it)
    logger "avahi-daemon.postconf:" "Removing system-generated $1 so avahi-daemon will quietly exit."
    mv $1 $1.system

I like seeing the entry in the log file reminding me I'm killing off avahi-daemon.  The 'mv' command just sets it aside; it has the same effect as removing it, but keeps it around in case there is a desire to see what the system created by default.

Like anything you read here, implement this change if it makes sense to you, just leave it alone if you're uncomfortable messing with it.   

*** 

Hopefully you found this article helpful.  
Questions/comments?  Look for me as 'latenitetech' on http://www.snbforums.com/forums/asuswrt-merlin.42/  
Glad to help if I can.

-Mark
