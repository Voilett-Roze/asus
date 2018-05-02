### Introduction

Asuswrt comes with built-in support for numerous tunnelling methods that allows you to get IPv6 support through your IPv4-only ISP.   You can use a 6in4 tunnel provided by Hurricane Electric.


### Hurricane Electric (AKA TunnelBroker) service

Signing up for a tunnel with them is free, and simple.  They will provide you with a routed /64 block (meaning you get your own static, fully routed, block of 2^64 IPs).  You can also request a larger block if needed (for example if you want to do some additional subnetting).

Setting up the router is fairly simple.  On the IPv6 section simply select the 6in4 option, and fill up the informations provided by HE:

* IPv4 remote Endpoint = Server IPv4 Address on TunnelBroker
* IPv6 client IP = Client IPv6 Address (without the /64 at the end - that is entered below)
* IPv6 Prefix Lenght will be 64
* IPv6 Prefix = Routed /64: (without the /64 at the end)
* Prefix Length will be 64
* IPv6 DNS Server 1 is optional (if your ISP's DNS support IPv6 resolution), otherwise enter the Anycasted IPv6 Caching Nameserver

**Router Settings page**  =  **HE  Sites setting page**
* Server IPv4 Address =  Use Server IPv4 Address: on Tunnel Details IPv6 Tunnel Tab
* Client IPv6 Address =  Use Client IPv6 Address: on Tunnel Details IPv6 Tunnel Tab
* IPv6 Prefix Length  =  Use 64
* Server IPv6 Address =  Use Server IPv6 Address: on Tunnel Details IPv6 Tunnel Tab
* Tunnel MTU 	      =  Use MTU  on Tunnel Details Advanced tab 
* Tunnel TTL 	      =  Router defaults are fine
* LAN Prefix Length   =  Use 48
* LAN IPv6 Prefix     =  Routed /48: hit button to  get your prefix xxxx:xxx:xxxx:: 


Also make sure to enable Router Advertisement.  This is what will automatically assign IPv6 addresses to your devices on your LAN, and provide them with routing information (a bit similar to what DHCP does for IPv4, but more advanced).

The next step: you need to somehow tell Hurricane Electric what your current IPv4 is.  There are two methods.

1) You can use the TunnelBroker option on the DDNS tab make account with Trial button in router. Just like a regular Dynamic DNS service, this will take care of letting HE know about any IP change.

**Setup On Router DDNS page**
* Host name                   = Use Tunnel ID : "xxxxxx" on Tunnel Details IPv6 Tunnel Tab
* User Name or E-mail Address = Use User ID "xxxxxxxxxxxxxxxx.xxxxxxxx" on Main page of HE upon login
* Password or DDNS Key        = Use Update Key on Tunnel Details Advanced tab 


2) If you want to use DDNS for a regular DDNS service, you can replace the functionality using a custom script.


### Configuring a tunnel endpoint update script

For the second method, you will need to enable the [JFFS](https://github.com/RMerl/asuswrt-merlin/wiki/JFFS) partition and custom configs and scripts.  Once that's done, create a `/jffs/scripts/wan-start` [user script](https://github.com/RMerl/asuswrt-merlin/wiki/User-scripts) that will take care of updating HE with your current WAN IP.  Here is an example script:

NOTE: If your tunnel was made before 19th Jan 2014, you need to uncomment the second wget line in the script, and comment the first. That's because old tunnels need your md5'd password, and newer ones need the tunnel update key. The detailed API is updating on [website](https://forums.he.net/index.php?topic=3153.0). Check it if anything goes wrong.

```
#!/bin/sh
#
# v1.40-rm3 Oct 31, 2017
# v1.41-gunnyst May 2, 2018
#

# ***************************
# settings
# ***************************

USERNAME="username used to log into your account"
PASSWORD="your 'Update Key' defined on the 'Advanced' tab (newer tunnels) or password (old tunnels)"
TUNNELID="your 'Tunnel ID' defined on the 'IPv6 Tunnel' tab"
TIMEOUT="10"
LOGFILE="/tmp/ipv6.log"


# ***************************
# advanced settings
# ***************************

# assuming your WAN is configured as 'ppp0' this will quickly and reliably give you your public WAN IP address
PUBLIC_IP=$(ip addr show ppp0 | awk '/inet /{gsub(/.*t/,"",$2);print$2}')

# accept PINGs from HE's endpoint verificiation server (or manually set public IP on https://www.tunnelbroker.net)
iptables -I INPUT 1 -s 66.220.2.74 -p icmp -j ACCEPT


# ***************************
# tunnel endpoing update
# ***************************

# update IP
if [ -z "${PUBLIC_IP}" ]; then
  wget --no-check-certificate --quiet --inet4-only --timeout=${TIMEOUT} -a - --output-document=${LOGFILE} "https://ipv4.tunnelbroker.net/nic/update?username=${USERNAME}&password=${PASSWORD}&hostname=${TUNNELID}" 2>&1
else
  wget --no-check-certificate --quiet --inet4-only --timeout=${TIMEOUT} -a - --output-document=${LOGFILE} "https://ipv4.tunnelbroker.net/nic/update?username=${USERNAME}&password=${PASSWORD}&hostname=${TUNNELID}&myip=${PUBLIC_IP}" 2>&1
fi

# update IP (for old tunnels)
# get a hash of the plaintext password
# MD5PASSWD=`echo -n ${PASSWORD} | md5sum | sed -e 's/  -//g'`
# if [ -z "${PUBLIC_IP}" ]; then
#   wget --no-check-certificate --quiet --inet4-only --timeout=${TIMEOUT} -a - --output-document=${LOGFILE} "https://ipv4.tunnelbroker.net/nic/update?username=${USERNAME}&password=${MD5PASSWD}&hostname=${TUNNELID}" 2>&1
# else
#   wget --no-check-certificate --quiet --inet4-only --timeout=${TIMEOUT} -a - --output-document=${LOGFILE} "https://ipv4.tunnelbroker.net/nic/update?username=${USERNAME}&password=${MD5PASSWD}&hostname=${TUNNELID}&myip=${PUBLIC_IP}" 2>&1
# fi
```

and don't forget to make it executable:
```
chmod +x /jffs/scripts/wan-start
```

Edit the first 3 parameters at the top: username, password (forolder tunnels)/update key (for newer tunnels) and tunnel ID. You can manually execute the script, then check if /tmp/ipv6.log contains your public IPv4 address to ensure that the script is running correctly.

After that, you can test your tunnel by accessing, for example, http://www.whatismyipv6.com.  If you see an IPv6 (meaning a long hexadecimal string), then congratulations - you're set!



### Conclusion

That pretty much covers it.  Tunnels are a great way for you to familiarize yourself with IPv6, regardless of what your ISP provides you.
