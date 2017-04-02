### Introduction
If you set the DDNS (dynamic DNS) service to "Custom", then you can fully control the update process through a `ddns-start` user script (which could launch a custom update client, or run a simple "wget" on a provider's update URL). The ddns-start script is passed the WAN IP as an argument.

Note that your custom script is responsible for notifying the firmware on the success or failure of the process.  To do this your script must execute:

```
/sbin/ddns_custom_updated 0 or 1
```
(where 0 = failure, 1 = successful update)

If you can't determine the success or failure, then report it as a success to ensure that the firmware won't continuously try to force an update.

Finally, like all [[user scripts|User-scripts]], the option to support custom scripts and config files must be enabled under Administration -> System.

After enabling custom scripts, place the contents of your update script in `/jffs/scripts/ddns-start` 

# Using a DDNS with VPN
Here is an example of a script for redirecting a DDNS to a VPN IP, in openvpn-event script, add:
```
sh /jffs/scripts/up.sh &
```
create /jffs/scripts/up.sh and add the following code, editing the username, password and dyndns host
```
#!/bin/sh
#keep looping until all the routing for the VPN tunnel is established
while [ ! -n  "`ifconfig | grep tun11`" ]; do
    sleep 1
done
#once established, get VPN IP
VPNIP=$(wget -qO - http://cfaj.freeshell.org/ipaddr.cgi)
sleep 10
ez-ipupdate -S dyndns -u user:password -h host.dyndns.org -a $VPNIP #update dyndns with VPN IP
exit 0
```
and make it executable
```
chmod 700 /jffs/scripts/up.sh
```

For more sample scripts please check the following [[link|DDNS-Sample-Scripts]]
