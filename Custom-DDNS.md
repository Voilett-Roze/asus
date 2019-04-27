>Note: _This might need to be revised following the switch to In-a-Dyn with version 384.7, this page could be merged with [[DDNS-services]]._

### Introduction
If you set the DDNS (dynamic DNS) service to "Custom", then you can fully control the update process through a `ddns-start` user script (which could launch a custom update client, or run a simple "wget" on a provider's update URL). The ddns-start script is passed the WAN IP as an argument.

Note that your custom script is responsible for notifying the firmware on the success or failure of the process.  To do this your script must execute:

```
/sbin/ddns_custom_updated 0 or 1
```
(where 0 = failure, 1 = successful update)

If you can't determine the success or failure, then report it as a success to ensure that the firmware won't continuously try to force an update.

Finally, like all [[user scripts|User-scripts]], the option to support custom scripts and config files must be enabled under Administration -> System.

After enabling custom scripts, place the contents of your update script in `/jffs/scripts/ddns-start` and Enable the DDNS Client in WAN -> DDNS and use `Custom` as Server.

# Using a DDNS with Double NAT
If your ASUS router is double NATed behind your ISP's router, you may need to
retrieve your external IP rather than using the one passed to it from the
Custom DDNS settings. Find the line in your update script where the IP is used
```
IP=${1}
```
and change it to get the IP from an external source
```
IP="$(curl -fs4 https://myip.dnsomatic.com/)"
```
The above uses dnsomatic, but it can be modified to work with any source. The OpenWrt wiki provides a list [here](https://openwrt.org/docs/guide-user/services/ddns/client#detecting_public_ip).

# Using a DDNS with VPN
Here is an example of a script for redirecting a DDNS to a VPN IP, in `/jffs/scripts/openvpn-event` add:
```
#!/bin/sh

if [ "$script_type" = "up" ] && [ "$vpn_interface" = "tun11" ]; then
	# OpenVPN tunnel won't open until openvpn-event script is finished, run the rest in a background shell
	(
		# Loop until VPN tunnel is established or the timeout limit is reached
		COUNTER=0
		LIMIT=10
		while [ "$COUNTER" -le "$LIMIT" ] && ! ifconfig | grep -Fq "tun11"; do
			sleep 1
			COUNTER=$((COUNTER + 1))
		done

		# Update your DDNS here, some examples below

		# Update dyndns via ez-ipupdate with VPN IP (pre 384.7)
		VPNIP="$(curl -fs4 http://myip.dnsomatic.com/)"
		ez-ipupdate -S "dyndns" -u "user:password" -h "host.dyndns.org" -a "$VPNIP"

		# Update via inadyn (384.7+)
		inadyn --once -f "/jffs/inadyn.conf"

		# Update via curl
		API="xxxxxxxxxxxxxxxx" # Your afraid.org API Key
		curl -fs -o /dev/null "https://freedns.afraid.org/dynamic/update.php?${API}"
	) &
fi
```
and make sure it's executable
```
chmod +x /jffs/scripts/openvpn-event
```

# Scripts for specific providers
Sample scripts for many DDNS providers and DNS services are available [[here|DDNS-Sample-Scripts]].
