Here is a simple guide for syncing your time from your router instead of from the internet

* Perform the following command `nano /jffs/scripts/services-start`
* append the following content

```
if [ "$(nvram get ntp_ready)" == "1" ];then  # Only start the NTP server if router has itself synchronised with Internet
   ntpd   -l
else
   logger -st "($(basename $0))" $$ "***ERROR Router cannot get initialise NTP Server!"
fi
```

then set point your devices that needs NTP service towards your routers IP

# 3rd Party Addons

There is a addon package for Asus-Merlin Firmware called NTP Daemon for ASUSWRT/Merlin that shows how much it syncs the NTP Daemon with graphs (see screenshot below) too learn more or get support on this please [click here](https://www.snbforums.com/threads/ntp-daemon-for-asuswrt-merlin.28041/)

![](http://oi67.tinypic.com/x39xd1.jpg)