Here is a simple guide for syncing your time from your router instead of from the internet

* Create directory `/jffs/ntp/` and create a suitable ntp.conf-file using command `nano /jffs/ntp/ntp.conf`. 
This is required to recreate the ntpd-config-file `/etc/ntp.conf` after every router-reboot (as ram is volatile).
* Example for ntp.conf:
```
server 0.de.pool.ntp.org iburst
server 1.de.pool.ntp.org iburst
server 2.de.pool.ntp.org iburst
```
* Perform the following command `nano /jffs/scripts/post-mount`
Please note that the rudimentary Busybox ntp-implementation knows almost no commands as restrict, driftfile, interface...
* append the following content

```
#!/bin/sh
if [ "$(nvram get ntp_ready)" == "1" ];
then  # Only start the NTP server if router has itself synchronised with Internet
   cp /jffs/ntp/ntp.conf /etc/
   logger -st "($(basename $0))" $$ "***SUCCESS Router copied ntp.conf-file to /etc!"
   ntpd   -l
   logger -st "($(basename $0))" $$ "***SUCCESS Router initialized NTP Server!"
else
   logger -st "($(basename $0))" $$ "***ERROR Router cannot get initialise NTP Server!"
fi
```

* then set point your devices that needs NTP service towards your routers IP.
* Don't forget to set the script as being executable: `chmod a+rx /jffs/scripts/*`


# 3rd Party Addons

There is a addon package for Asus-Merlin Firmware called NTP Daemon for ASUSWRT/Merlin that shows how much it syncs the NTP Daemon with graphs (see screenshot below) too learn more or get support on this please [click here](https://www.snbforums.com/threads/ntp-daemon-for-asuswrt-merlin.28041/)

![](http://oi67.tinypic.com/x39xd1.jpg)