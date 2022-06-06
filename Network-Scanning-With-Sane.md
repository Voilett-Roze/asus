This page describes how to enable Network Scanning using a USB scanner or multi-function printer in Merlin using S.A.N.E. (Scan Access Now Easy).

### Requirements:
* A router running Merlin duh...
* USB Stick with Entware configured [instructions](https://github.com/RMerl/asuswrt-merlin.ng/wiki/Entware)
* A USB Scanner or Multi-Function Printer
* Router accessible via ssh
* Familiarity with ssh access and command line tools line vi

### Router Setup Instructions:
* Ssh to router and type 
```
opkg install sane-backends sane-frontends sane-libs dbus xinetd
```
* Create file `/opt/etc/xinetd.d/saned` and put the following content:  
```
service saned
{
    type = UNLISTED
    port = 6566
    socket_type = stream
    server = /opt/sbin/saned
    protocol = tcp
    user = <REPLACE HERE WITH YOUR ADMIN USERNAME>
    group = root
    wait = no
    disable = no
}
```
* Edit file `/opt/share/dbus-1/system.conf` and replace the following line:  
**`<user>__REPLACE HERE WITH YOUR ADMIN USERNAME__</user>`**  
with  
**`<user>admin</user>`**  
* Edit file `/opt/etc/sane.d/saned.conf` and add your subnet to allow access, for example: 192.168.1.0/24  
* Reboot router and make sure that dbus and xinetd are running (this step might not be required if you know how to start the services manually from `/opt/etc/init.d`)

NOTE: should you experience the service dbus not coming up after router reboot. Simply add the following lines to the file `/opt/etc/init.d/S20dbus` before the line `. /opt/etc/init.d/rc.func`:  
```
if test -f /opt/var/run/dbus.pid; then
        rm /opt/var/run/dbus.pid
fi
```
### Special Hardware Instructions
### HP:
* Ssh to router and type the command: `opkg --force-overwrite install hplip`

### Test functionality:
* Type: `scanimage -L`, your device should be displayed
* Type: `scanimage --test`, your device should start testing and display status

### On The Client Side (Linux):
* Install the sane package. This changes by distribution
* Edit file `/etc/sane.d/net.conf` and add the ip or hostname of the router
* Test the same way with the `scanimage` utility

### On The Client Side (Windows):
* Download SaneTwain [http://sanetwain.ozuzo.net](http://sanetwain.ozuzo.net)
* On start-up put the router's ip/host

Suggestions are always welcomed.
 