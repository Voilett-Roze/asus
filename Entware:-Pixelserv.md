* Work in progress
* If possible define verification (test) criteria for each step


### ARM Pixelserv Installation Process
1. Follow the official installation steps at: https://github.com/kvic-z/pixelserv-tls
2. Get the Mips or ARM release from Pixelserv: https://github.com/kvic-z/pixelserv-tls/releases
3. Make Pixelserv autostart on boot place this file at /opt/etc/init.d/S81pixelserv-custom
```#!/bin/sh

ENABLED=yes
PROCS=pixelserv
ARGS="`ifconfig br0 | awk '/inet addr/{print substr($2,6)}'` -p 80 -p 8080 -k 443 -k 2443 -u $USER"
PREARGS=""
DESC=$PROCS
PATH=/opt/sbin:/opt/bin:/opt/usr/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin

. /opt/etc/init.d/rc.func```

4. for possible issues use this thread [Source: SNB forums](http://www.snbforums.com/threads/pixelserv-a-better-one-pixel-webserver-for-adblock.26114/)
5. Use blocklists like [Entware Adblock Solution](https://github.com/Entware-ng/Entware-ng/wiki/Using-AdBlock--filters) or [uBlockr](https://gitlab.com/spitfire-project/ublockr)
