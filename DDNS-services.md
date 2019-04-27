## Introduction to In-a-dyn
Starting with version 384.7, Asuswrt-Merlin uses [In-a-dyn](https://github.com/troglobit/inadyn) for DDNS updates instead of the obsolete ez-ipupdate client used until then.  This client adds HTTPS support, in addition to supporting more services, and being able to easily be customized to support a lot of additional services

By default, the same services as before are supported (including full support for Asus's own DDNS through a custom-developed plugin).  Freedns.afraid.org was also added to the list of natively supported services on the webui.  You can further extend In-a-dyn either by adding additional config entries through the usual methods (`inadyn.conf`, `inadyn.conf.add` and `inadyn.postconf`), in addition to the previously supported `ddns-start` script.  Note that if your ddns-start script was previously using ez-ipupdate, you will have to rewrite it to use In-a-dyn instead.


## Using one of the services supported by In-a-dyn but not by the Asuswrt-Merlin webui
First, create an `inadyn.conf` config, and store it into `/jffs/` so it will persist through reboots (not to be confused with `/jffs/configs/inadyn.conf`, which overwrites WebUI settings).  Please consult the In-a-dyn [documentation](https://github.com/troglobit/inadyn) for more information on how to configure an In-a-dyn service.

Here is an example for a complete In-a-dyn config file for a custom service, based after selfhost.de:

```
custom selfhost {
	ddns-server = carol.selfhost.de
	ddns-path = "/nic/update?hostname=%%h&myip=%%i"
	hostname = MY_HOSTNAME.selfhost.eu
	username    = 12345
	password    = "MY_PASSWORD"
}
```

If you wish to use your local WAN IP instead of relying on the DDNS service's remote check method, you can insert the following line within your custom or provider block:

```
checkip-command = "/usr/sbin/nvram get wan0_ipaddr"
```

Verify the configuration using:

```
inadyn --check-config -f "/jffs/inadyn.conf"
```

Next, create a `ddns-start` script that will invoke In-a-dyn, pointed at your custom config. The script must be placed in `/jffs/scripts/`. The `--once` option will ensure that In-a-dyn will exit once it has completed its update (Asuswrt-Merlin runs In-a-dyn as a client rather than as a daemon). Such a script would look like this:

```
#!/bin/sh
inadyn --once -f "/jffs/inadyn.conf" -e "/sbin/ddns_custom_updated 1" --exec-nochg "/sbin/ddns_custom_updated 1"
```

Give the `ddns-start` execute permissions:

```
chmod +x /jffs/scripts/ddns-start
```

Note In-a-dyn will take care of calling ddns_updated as appropriate if the update succeeded when using these parameters, so no need to explicitly run it manually after the update.

On the WebUI, set your DDNS provider to "Custom" and provide a hostname. Apply the changes and you should see a `Registration is successful` dialog.


## Updating multiple services
In-a-dyn allows you to define multiple services within a single config file.  If for example you configured no-ip.com through the WebUI, and you also want to update freedns.afraid.org, create the following `/jffs/configs/inadyn.conf.add` file, with your second service definition:

```
provider default@freedns.afraid.org {
	hostname = MY_HOSTNAME.mooo.com
	username = "MY_USERNAME"
	password = "MY_PASSWORD"
	checkip-command = "/usr/sbin/nvram get wan0_ipaddr"
}
```

Whenever Asuswrt-Merlin will issue a ddns update, both the service on the WebUI and your additional Freedns service will be updated at the same time.

Note that the "Custom" and "www.oray.com" providers do not use In-a-dyn, therefore `/jffs/configs/inadyn.conf`, `/jffs/configs/inadyn.conf.add` and `/jffs/scripts/inadyn.postconf` will also be unused. In-a-dyn can be called via `ddns-start` as above in these cases.


## DDNS updates through either double NAT or a CGNAT connection
Also starting with 384.7, Asuswrt-Merlin lets you choose between **Internal** or **External** IP check methods.  The **Internal** method is the same one that was used until now - the IP on your WAN interface is retrieved from nvram, and provided to In-a-dyn, which will provide it to the DDNS server (provided that server supports it).  The **External** method is new, and lets the DDNS service provider determine your WAN IP (based on the IP from which you connect with it).  This will allow your provider to retrieve your real public IP if your Asus router is behind another router, or your ISP uses CGNAT to provide you with a non-public IP.

Note that this parameter only applies to the built-in services listed on the WebUI.  If you use a custom config or script, it will be up to you to handle how the IP is provided - that WebUI setting will have no effect.