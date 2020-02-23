## Introduction ##

This guide will help you implement Selective **Port** (or **MAC address**) routing, via the [VPN](http://en.wikipedia.org/wiki/Virtual_private_network) (or Selectively Route Ports via the WAN  - local [ISP](http://en.wikipedia.org/wiki/Internet_service_provider))


## Prerequisites ##

* An Asuswrt-Merlin compatible ARM-based router with Asuswrt-Merlin **v380.xx** or later
* [JFFS](https://github.com/RMerl/asuswrt-merlin.ng/wiki/JFFS) partion enabled and formatted
* A working VPN on your router (tested manually making sure the VPN works)
* A [winSCP](http://winscp.net/eng/download.php#download2) client setup with SSH enabled on your ROUTER
* [Notepad ++](http://notepad-plus-plus.org/)

## Installation ##

Policy routing of LAN devices/IPs/CIDRs or target IPs/CIDRs is available via the GUI, but the firmware does not include Routing Policy Data Base (**RPDB**) fwmark rules.

It is recommended that you use the following RPDB fwmarks for the Selective Port routing
```
ip rule

0:	from all lookup local
9990:	from all fwmark 0x8000/0x8000 lookup main
9991:	from all fwmark 0x7000/0x7000 lookup ovpnc4
9992:	from all fwmark 0x3000/0x3000 lookup ovpnc5
9993:	from all fwmark 0x1000/0x1000 lookup ovpnc1
9994:	from all fwmark 0x2000/0x2000 lookup ovpnc2
9995:	from all fwmark 0x4000/0x4000 lookup ovpnc3
32766:  from all lookup main
32767:  from all lookup default
```
The **RPDB** fwmark rules should be created by script **/jffs/scripts/nat-start** (see Wki entry [User scripts](https://github.com/RMerl/asuswrt-merlin.ng/wiki/User-scripts) if **nat-start** doesn't exist)

```
#!/bin/sh

sleep 10  # During the boot process nat-start may run multiple times so this is required               

# Ensure duplicate rules are not created
for VPN_ID in 0 1 2 3 4 5
   do
      ip rule del prio 999$VPN_ID  2>/dev/null
   done

# Create the RPDB rules
ip rule add from 0/0 fwmark "0x8000/0x8000" table main   prio 9990        # WAN   fwmark
ip rule add from 0/0 fwmark "0x7000/0x7000" table ovpnc4 prio 9991        # VPN 4 fwmark
ip rule add from 0/0 fwmark "0x3000/0x3000" table ovpnc5 prio 9992        # VPN 5 fwmark
ip rule add from 0/0 fwmark "0x1000/0x1000" table ovpnc1 prio 9993        # VPN 1 fwmark
ip rule add from 0/0 fwmark "0x2000/0x2000" table ovpnc2 prio 9994        # VPN 2 fwmark
ip rule add from 0/0 fwmark "0x4000/0x4000" table ovpnc3 prio 9995        # VPN 3 fwmark
```
or they can be added on demand when the appropriate VPN Client is started, and deleted when the VPN Client is stopped.
(see openvpn-event triggers _vpnclientX-route-pre-up/vpnclientX-down_)

Once the RPDB fwmarks are defined/ACTIVE, it is a simple case of adding (to **nat-start** or the _openvpn-event_ **route-up** trigger etc.) the appropriate iptables rule to Selectively route the desired Ports via the designated VPN Client.

***Example 1.***

Selectively route **Web HTTP/HTTPS** (**Port 80** and **443**) traffic from **192.168.1.99** via **VPN Client 2**
```
iptables -t mangle -A PREROUTING -i br0 -m iprange --src-range 192.168.1.99 -p tcp -m multiport --dport 80,443 -j MARK --set-mark 0x2000/0x2000
```
***Example 2.***

Suppose **ALL** traffic from LAN device **192.168.1.88** is routed via a **VPN** but hosts the **RDP** service (**Port 3389**)

```
iptables -t mangle -A PREROUTING -i br0 -m iprange --src-range 192.168.1.88 -p tcp -m multiport --sport 3389 -j MARK --set-mark 0x8000/0x8000
```
_**NOTE:**_ To allow access inbound from the **WAN** you will also need to ensure that **Port 3389** is forwarded in the _**WAN** - **Virtual Server** / **Port Forwarding** GUI_.

***

_Port FORWARD **without** using the GUI_

e.g. Expose say **Port 63389** to the **WAN**, which is actually **RDP Port 3389** internally, hosted on **192.168.1.88**
```
iptables -t nat -D VSERVER -p tcp -m tcp --dport 63389 -j DNAT --to-destination 192.168.1.88:3389 2> /dev/null
iptables -t nat -I VSERVER -p tcp -m tcp --dport 63389 -j DNAT --to-destination 192.168.1.88:3389
```

***


***Example 3.***

You can also specify a range of ports and even combine the Selective Port Routing with multiple source/destinations etc.

Ten LAN devices (**192.168.1.100** to **192.168.1.109** inclusive) will Selectively Route thirteen ports (**80,443** and **54000** to **54010** inclusive) via **VPN Client 3** 

```
iptables -t mangle -A PREROUTING -i br0 -m iprange --src-range 192.168.1.100-192.168.1.109 -p tcp -m multiport --dport 80,443,54000:54010 -j MARK --set-mark 0x4000/0x4000
```

***Example 4.***

Services hosted on the router may also be routed via the **VPN**, replace **X** with the **VPN** instance, and substitute **--sport xxxx/--dport yyyy** as required by the application.

```
iptables -t mangle -A OUTPUT -p tcp -m multiport --sport xxxx  -j MARK --set-mark 0xX000/0xX000
iptables -t nat -A POSTROUTING -s $(nvram get wan0_ipaddr) -o tun1X -j MASQUERADE
```

***Example 5.***

Selectively route **Web HTTP/HTTPS** (**Port 80** and **443**) traffic from **MAC address** **xx:xx:xx:xx:xx:xx** via **VPN Client 2**
```
iptables -t mangle -A PREROUTING -i br0 -m mac --mac-source xx:xx:xx:xx:xx:xx -p tcp -m multiport --dport 80,443 -j MARK --set-mark 0x2000/0x2000
```

However, the use of IPSETs can greatly enhance the RPDB fwmark Selective Routing method, both by performance and flexibility.

e.g. A single IPSET Selective Routing rule can reference (thousands) of Source/Destinations IPs,Ports,MACs and Domains.

```
iptables -t mangle -A PREROUTING -i br0 -m set --match-set VPN1 src,src,dst -j MARK --set-xmark 0x1000/0x1000
```
where IPSET VPN1 could contain multiple IPSETS i.e. a 'Ports' only IPSET,a Source IP/CIDR,Destination Port IPSET, and a Source MAC IPSET etc. 

NOTE: [Small Netbuilder member @Xentrk](https://www.snbforums.com/members/xentrk.49161/) makes use of the IPSET technique for the Selective Routing of domains such as **Netflix**, **Hulu**, **BBC** etc.

e.g. [Selective Routing of Netflix](https://www.snbforums.com/threads/selective-routing-for-netflix.42661/)

