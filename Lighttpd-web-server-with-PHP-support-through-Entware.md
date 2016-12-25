First, setup Entware from [this](https://github.com/RMerl/asuswrt-merlin/wiki/Entware) guide.

Login to router with a terminal like putty and enter this commands:
### The following _tinyurl.com_ links are no longer active. Please update this tutorial with working links/source. Better yet, if someone can explain each step in more detail, it would greatly help n00bs understand what's going on (more than copypasta instructions)!

Since the links are deprecated, this URL helps with customization of the server. [https://wiki.openwrt.org/doc/howto/http.lighttpd](https://wiki.openwrt.org/doc/howto/http.lighttpd)
  
```
opkg install lighttpd php5-cgi lighttpd-mod-fastcgi
rm /opt/etc/lighttpd/lighttpd.conf
wget -c -O /opt/etc/lighttpd/lighttpd.conf http://tinyurl.com/amvkxt3 --no-check-certificate
wget -c -O /opt/share/www/index.html http://tinyurl.com/bxfxpq6 --no-check-certificate
wget -c -O /opt/share/www/test.php http://tinyurl.com/b9b34kp --no-check-certificate
/opt/etc/init.d/S80lighttpd start
```
Go to [router.asus.com:81](http://router.asus.com:81) and if you see this page, the lighttpd web server is configured correctly

![if](http://i47.tinypic.com/rm5it1.png)

Go to [router.asus.com:81/test.php](http://router.asus.com:81/test.php) and if you see this page, the php-mod-fastcgi is configured correctly

![php](http://i50.tinypic.com/i5usfo.png)

###TO ACCESS THE WEBSITE FROM WAN
```
opkg install nano
nano /jffs/scripts/firewall-start
```

Paste this lines
```
#!/bin/sh
iptables -I INPUT -p tcp --destination-port 81 -j ACCEPT
```

Save with **CTRL-O** / **Enter** / and exit nano with **CTRL-X**

Give right permission
```
chmod a+rx /jffs/scripts/firewall-start
```

Go to [Port forwarding page](router.asus.com/Advanced_VirtualServer_Content.asp) and redirect port 80 to 81, after reboot you should have access from wan.

![wan](http://i47.tinypic.com/309hgqr.png)

Youtube video [here](http://youtu.be/KHABSd7qB2M)

Post issues [here](https://www.hqt.ro/lighttpd-web-server-with-php-support-through-entware-ng/)