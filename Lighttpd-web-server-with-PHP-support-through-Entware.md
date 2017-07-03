First, setup Entware from [github RMerl asuswrt-merlin guide](https://github.com/RMerl/asuswrt-merlin/wiki/Entware).

Login to router with a terminal like putty and enter the commands below.

Additional [the openwrt.org wiki helps with customization of the server](https://wiki.openwrt.org/doc/howto/http.lighttpd).
  
```
opkg install lighttpd php5-cgi lighttpd-mod-fastcgi
```

### Choose port 81 as lighttpd server port and fix the uploads directory.
```
sed -i 's/#server.port                 = 81/server.port                 = 81/g' "/opt/etc/lighttpd/lighttpd.conf"
sed -i "/server.upload-dirs*/cserver.upload-dirs          = ( \"/opt/tmp\" )" "/opt/etc/lighttpd/lighttpd.conf"
```
or
```
# nano /opt/etc/lighttpd/lighttpd.conf
```
and remove the commenting hash (#) prefix from the `server.port` line,
and change the line that starts with `server.upload-dirs` from `/tmp` to `/opt/tmp`.

### Create a plain html test page
```
cat >> /opt/share/www/index.html << EOF
<html>
<head>
<title>lighttpd default page</title>
</head>
<body>
<h2>lighttpd server is running.</h2>
</body>
</html>
EOF
```
or
```
# nano /opt/share/www/index.html
```
and copy and paste the html source code lines from the section above.

### Create a php dynamic test page
```
cat >> /opt/share/www/test.php << EOF
<?php
phpinfo();
?>
EOF
```
or by using your favourite text editor:
```
# nano /opt/share/www/test.php
```
by copying and pasting only the php source code section from above.

### Finally start lighttpd
```
/opt/etc/init.d/S80lighttpd start
```
Go to [router.asus.com:81](http://router.asus.com:81) and if you see this page, the lighttpd web server is configured correctly:

> 
> lighttpd server is running.
> 

Go to [router.asus.com:81/test.php](http://router.asus.com:81/test.php) and if you see this page, the php-mod-fastcgi is configured correctly

![php](http://i50.tinypic.com/i5usfo.png)

### To access the website from WAN
```
opkg install nano
nano /jffs/scripts/firewall-start
```

Paste these lines
```
#!/bin/sh
iptables -I INPUT -p tcp --destination-port 81 -j ACCEPT
```

Save with <kbd>Ctrl</kbd>+<kbd>O</kbd>, <kbd>Enter</kbd>, and exit nano with <kbd>Ctrl</kbd>+<kbd>X</kbd>

Give it firewall file the correct file permissions
```
chmod a+rx /jffs/scripts/firewall-start
```

Go to [Port forwarding page](router.asus.com/Advanced_VirtualServer_Content.asp) and redirect port 80 to 81, after reboot you should have access from wan.

![wan](http://i47.tinypic.com/309hgqr.png)

Youtube video [here](http://youtu.be/KHABSd7qB2M)

Post issues [here](https://www.hqt.ro/lighttpd-web-server-with-php-support-through-entware-ng/)