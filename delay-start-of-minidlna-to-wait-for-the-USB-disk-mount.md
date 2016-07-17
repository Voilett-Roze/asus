When the USB disk is getting to slow mounted, you can use the following script to delay the minidlna start do not screw the mount points. Put following content to `/jffs/scripts/post-mount`:

```
#!/bin/sh

mountpoint=/tmp/mnt/DISK1
# the disk you use for media server

dlna_config=/tmp/mnt/DISK1/Scripts/minidlna.conf
# the configuration file created for minidlna to use the custom settings.


if [ $1 = $mountpoint ]
then
  /usr/sbin/minidlna -f $dlna_config -R
fi
```

The `minidlna.conf` I took from the original provided from Asus like in example this:

```
network_interface=br0
port=8200
friendly_name=RT-AC56U
db_dir=/tmp/mnt/DISK1/Router
enable_tivo=no
strict_dlna=no
presentation_url=http://192.168.102.1:80/
inotify=yes
notify_interval=600
album_art_names=Cover.jpg/cover.jpg/Thumb.jpg/thumb.jpg
media_dir=/tmp/mnt/DISK1/Media
serial=d8:50:e6:50:09:c0
model_number=3.0.0.4.374.38
```

so the script will load this configuration file and load minidlna with the above settings. 

**IMPORTANT: you need to disable the Media Server in the GUI interface to work proper. **

Don't forget to make script executable:

```
chmod +x /jffs/scripts/post-mount
```

Reboot router to take effect. 

# Alternate method
for systems without /jffs

--added by pliu1s (I'm a newbie so I'm not sure where to put this)

Another way to do this in Archlinux is by using systemctl and fstab options.  The basic idea is for fstab to use x-systemd.automount and then to make x-systemd.automount an "After=" requirement for minidlna.  This way the kernel holds off minidlna until fsck is done.  The detailed steps are:

1. set up /etc/fstab to use x-systemd.automount, such as:

>    /dev/sdb1 /media/usb    ext4 defaults,nofail,x-systemd.automount,x-systemd.device-timeout=1 0 2


2. add 1 line under the [Unit] section of /etc/systemd/system/multi-user.target.wants/minidlna.service

>     [Unit]
>     Description=minidlna server
>     After=network.target x-systemd.automount


I found the straightforward way of making the mount point a requirement for minidlna didn't work because though the USB disk would get mounted, the directories inside the mount point were not 'revealed' by the OS until fsck was done.  But since fsck would take a very long time, minidlna would try to access the directory before fsck was done, get an error that the directory doesn't exist, and then delete that directory from its database.  The end result was that minidlna would start successfully, but the database would be wiped clean.  The above method using x-systemd.automount prevents this from happening.

Hope this helps those who come across this page as I did.> 