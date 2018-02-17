## Introduction and Prerequisites
This tutorial was developed on an Asus RT-AC68U with AsusWRT Merlin firmware, but could be adapted to every router that uses Entware repo (opkg) with little modification.
This guide is for users who already have a little experience with Linux system.

You need: A router with ENTWARE repository activated and an hard-drive attached with your Calibre library, but if you are reading this tutorial probably you already have them.
At this moment (Jan 2018) Optware has a too old version of the php engine, so you need to change and go to Entware.

## Let's Start
Install and update ENTWARE. ( ASUS WRT Merlin: https://github.com/RMerl/asuswrt-merlin/wiki/Entware , general: https://github.com/Entware-ng/Entware-ng/wiki )

`opkg update`

## Prepare the Webserver
If you already have a webserver running, check if you have every package. 99% problems that you could bump into are produced by a missing package.

Install a webserver, I choose lighttpd, PHP7, and the other packages needed. The official guide of COPS doesn't report every package needed because was written for Debian Distribution witch inglobate many libraries in the "common", so you need some "extra" packages. 
Should work also with the php built in webserver ( https://github.com/seblucas/cops/wiki/Howto---PhpEmbeddedServer , but at this moment isn't available via opkg )

`opkg install php7 php7-fastcgi php7-mod-ctype php7-mod-dom php7-mod-gd php7-mod-intl php7-mod-json php7-mod-mbstring php7-mod-mcrypt php7-mod-pdo php7-mod-pdo-sqlite php7-mod-simplexml php7-mod-sqlite3 php7-mod-xml libxml2 php7-mod-fileinfo lighttpd lighttpd-mod-fastcgi libsqlite3 php7-mod-xmlwriter`

(Probably opkg will install some other packages as dependency for this)

Configure lighttpd : (from: https://github.com/RMerl/asuswrt-merlin/wiki/Lighttpd-web-server-with-PHP-support-through-Entware )


`sed -i 's/#server.port                 = 81/server.port                 = 81/g' "/opt/etc/lighttpd/lighttpd.conf"`
`sed -i "/server.upload-dirs*/cserver.upload-dirs          = ( \"/opt/tmp\" )" "/opt/etc/lighttpd/lighttpd.conf"`

(For OpenWRT path see this: https://wiki.openwrt.org/doc/howto/http.lighttpd )

And, finally, get PHP Working:


`cat >> /opt/etc/lighttpd/conf.d/30-fastcgi.conf  << EOF
server.modules += ( "mod_scgi" )
scgi.server = (
  "/RPC2" =>
    ( "127.0.0.1" =>
     (
        "socket" => "/opt/var/rpc.socket",
        "check-local" => "disable"
      )
    )
)

server.modules += ( "mod_fastcgi" )
fastcgi.server = (
  ".php" =>
    ( "localhost" =>
      ( "socket" => "/tmp/php-fcgi.sock",
        "bin-path" => "/opt/bin/php-fcgi",
        "max-procs" => 1,
        "bin-environment" =>
          ( "PHP_FCGI_CHILDREN" => "1",
             "PHP_FCGI_MAX_REQUESTS" => "100"
          )
      )
    )
)

server.port = 81

EOF`

(Again, for OpenWRT and others adapt path)
If you have performance problem reduce PHP_FCGI_MAX_REQUESTS and PHP_FCGI_CHILDREN values.  

## Install COPS

Get cops (example code for the 1.0.1 version):

`cd /opt/share/www`
`wget https://github.com/seblucas/cops/releases/download/1.0.1/cops-1.0.1.zip`
`mkdir cops`
`unzip cops-1.0.1.zip -d ./cops`

###URL Rewrite for KOBO:
Adding mod-rewriter (useful for Kobo users) (from: https://github.com/seblucas/cops/wiki/Url-Rewriting-with-COPS ):

`nano -w /opt/etc/lighttpd/conf.d/10-cops_rewrite.conf`

With something like this (it's only one row), in my installation I put cops in /opt/var/share/cops/ (witch will be http://router.asus.com:81/cops) :

`url.rewrite-once = ("/cops/download/(.*)/.*\.(.*)$" => "/cops/fetch.php?data=&type=", "^/cops/download/(\d+)/(\d+)/.*\.(.*)$" => "/cops/fetch.php?data=&db=&type=")`

## Configure COPS:

`nano -w /opt/share/www/cops/config_local.php`

put the absolute path of your calibre db in $config['calibre_directory'] , for example:

`<?php`
`$config['calibre_directory'] = '/tmp/mnt/NAS/calibre/BiblioNAS/';`
`?>`

For all available variables, check the file config_local.php.example (and config_default.php), for every one you want to modify set it into config_local.php at your desired value, otherwise will lose the mod at the first update of COPS!

### URL Rewrite for KOBO:
If you want direct-download using Kobo you should also set $config['cops_use_url_rewriting'] to 1 (need for automatic download from your ebook-reader using onboard browser), put in the row before ?>

`$config['cops_use_url_rewriting'] = "1";`

and $config['cops_provide_kepub'] to 1 to have the ebook recognized as a Kepub (chaptered paging, statistics, ...), again in the row before ?> add 

`$config['cops_provide_kepub'] = "1";`

## Ready? Go!
Let's start the server!

`/opt/etc/init.d/S80lighttpd restart`

(again, example is for AsusWRT Merlin, correct to your path if you have another firmware)

Go to your router ip on the port you choose, such as; http://router.asus.com:81/cops/checkconfig.php , everything should be marked as "OK"
![](https://www.snbforums.com/attachments/cops-ok-png.11428/)

## Finally consideration
For standard use (access, searching, reading and download) COPS is good, also from Kindle.
Remember to Stop the webserver to prevent users to work on the DB while it's open in Calibre!
1) Stop lighttpd server
2) Mount the postion on your PC using NFS (with nolock option), or with SMB if NFS is not available (NFS is faster and safe, f.e. you could allow only one IP to connect to that share, to prevent multiple open)
3) Add the library to Calibre
4) Do your work using Calibre
5) Disconnect Calibre from the library.
6) umount the share.
6) Lunch lighttpd.

## Will slow my router, my LAN, etc.?
It's possible. It depends on your hardware, weight of the library, how many users will access at the same time, etc. 
On my RT-AC68U I have not encountered any problems.
My target was to consult and download eBooks directly from my Kindle without having to use a PC. I got it. When I add o make high-management of ebook I use Calibre on NFS.

If you need in COPS it's possible to configure something about thumbnail generation and handling, to improve performance, listing, etc. 
Adjust to your needs this lines in cops/config_local.php could improve performance. This is mine, they're a little CPU aggressive...configure on your needs!

`    /*
     * Update Epub metadata before download
     * 1 : Yes (enable)
     * 0 : No
     */
    $config['cops_update_epub-metadata'] = "1";

    /*
     * Thumbnails are generated on-the-fly so it can be problematic on servers with slow CPU (Raspberry Pi, Dockstar, Piratebox, ...).
     * This configuration item allow to customize how thumbnail will be generated
     * "" : Generate thumbnail (CPU hungry)
     * "1" : always send the full size image (Network hungry)
     * any url : Send a constant image as the thumbnail (you can try "images/bookcover.png")
     */
    $config['cops_thumbnail_handling'] = "1";

    /*
     * Directory to keep resized thumbnails: allow to resize thumbnails only on first access, then use this cache.
     * $config['cops_thumbnail_handling'] must be ""
     * "" : don't cache thumbnail
     * "/tmp/cache/" (example) : will generate thumbnails in /tmp/cache/
     * BEWARE : it has to end with a /
     */
    $config['cops_thumbnail_cache_directory'] = "";
    /*
     * Max number of items per page
     * -1 unlimited
     */
    $config['cops_max_item_per_page'] = "-1";

    /*
     * Number of recent books to show
     */
    $config['cops_recentbooks_limit'] = '50';

`
In webserver you could work on max contemporary access and call (see lighttpd man page), p.e. reducing PHP_FCGI_MAX_REQUESTS values in /opt/etc/lighttpd/conf.d/30-fastcgi.conf.