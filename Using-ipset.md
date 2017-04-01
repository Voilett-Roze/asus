# Using ipset

Since 3.0.0.4_270.26 [ipset](http://en.wikipedia.org/wiki/Netfilter#ipset) feature has been implemented. This is a Netfilter extension which should be able to:
* store multiple IP addresses and/or port numbers and match against a filter list using iptables;
* dynamically update iptables rules against IP addresses or ports without a significant performance penalty
* express complex IP address and port based rulesets with one single iptables rule and benefit from the speed of IP sets.

# Ipset Version and Router Models

Newer router has [ipset version 6](http://ipset.netfilter.org/ipset.man.html) while older routers has ipset version 4. 

_Consult the chart below to see your ipset version._

| `Routers`    |`Ipset 4`|`Ipset 6`|
|--------------|:-------:|:-------:|
| `RT-N66U`    | x       |         |
| `RT-AC56U`   |         | x       |
| `RT-AC66U`   | x       |         |
| `RT-AC66U_B1`|         |         |
| `RT-AC68U`   |         | x       |
| `RT-AC68P`   |         | x       |
| `RT-AC68UF`  |         | x       |
| `RT-AC87U`   |         | x       |
| `RT-AC88U`   |         | x       |
| `RT-AC1750`  |         |         |
| `RT-AC1900`  |         |         |
| `RT-AC1900P` |         |         |
| `RT-AC3100`  |         | x       |
| `RT-AC3200`  |         | x       |
| `RT-AC5300`  |         | x       |

There is a full list of script that are maintained by users, most of the scripts are have various functions for blocking connections please read the description carefully before installing any of these scripts, not all scripts have maintainers and getting support on those scripts can be tricky.

> Note: The script with maintainers are linked in the chart to their respective support thread on snbforums if you have questions or suggestions.

| `Scriptname` |`Ipset 4`|`Ipset 6`|`Maintained by`|`Supports other platforms`|Description|
|--------------|:-------:|:-------:|:-------------:|:------------------------:|:----------:
|[Tor and Countries Block](https://www.snbforums.com/threads/country-blocking-script.36732/#post-311248)|x|x|redhat27|no|Block Tor or Countries| 
|[iblocklist-loader](https://www.snbforums.com/threads/iblocklist-com-generic-ipset-loader-for-ipset-v6-and-v4.37976/)|x|x|redhat27|yes|Block or allow using any list from iblocklist| 
|[Malware Filter](https://www.snbforums.com/threads/malware-filter-bad-host-ipset.35423/)|x|x|swetoast|yes|Malware Blocking|
|[Privacy Filter](https://www.snbforums.com/threads/privacy-filter-another-ipset-script.36801/)|x|x|swetoast|yes|Privacy|
|[Peerguardian v1](https://github.com/RMerl/asuswrt-merlin/wiki/Peerguardian-Scripts#peer-guardian)|x| | |no|Peerguardian|
|[Peerguardian v2](https://github.com/RMerl/asuswrt-merlin/wiki/Peerguardian-Scripts#peer-guardian-v2)|x| | |no|Peerguardian|
|[Peerguardian v3](https://github.com/RMerl/asuswrt-merlin/wiki/Peerguardian-Scripts#peer-guardian-v3)|x| | |no|Peerguardian|

# Tor and Countries Block 
Supports both IPSET 4 and 6

This is an example of using [ipset utility](http://manpages.ubuntu.com/manpages/lucid/man8/ipset.8.html) with two different set types: iphash and nethash. The example shows how to block incoming connection from [Tor](https://www.torproject.org/) nodes (iphash set type — number of ip addresses) and how to block incoming connection from whole countries (nethash set type - number of ip subnets). 

* Enable and format [JFFS](https://github.com/RMerl/asuswrt-merlin/wiki/JFFS) through WEB UI first,

* Then place [**this content**](https://raw.githubusercontent.com/shounak-de/iblocklist-loader/master/create-ipset-lists.sh) to `/jffs/scripts/create-ipset-lists.sh`

* Then make it executable:
```
chmod +x /jffs/scripts/create-ipset-lists.sh
```
* Finally call this at the end of your existing /jffs/firewall-start:
```
# Load ipset filter rules
sh /jffs/scripts/create-ipset-lists.sh
```

You may also run `/jffs/scripts/create-ipset-lists.sh` from command line or reboot router to apply new blocking rules immediately. Note that if you change the script country list or any other part, it would need a router reboot to take effect: _There are checks to prevent it from reloading the sets if the sets are already existing._

You can create a handy alias in your profile (in /opt/etc/profile or /jffs/configs/profile.add)

If you have ipset v4:
```
alias blockstats='iptables -L -v | grep " set"'
```
If you have ipset v6:
```
alias blockstats='iptables -L -v | grep "match-set"; ip6tables -L -v | grep "match-set"'
```
Then you can just issue 'blockstats' from the command prompt to see how well your blocklists are doing (see blocked packet count and byte count)

Note that every time you do something on the web UI or through your [android app](https://play.google.com/store/apps/details?id=com.asus.aihome) to control your router _that affects reloading the firewall rules_, `/jffs/scripts/firewall-start` will be called, so the iptables rules that are defined outside will be wiped out. To reinstate the rules as defined by this script, you'd need to add this to your _existing_ `/jffs/scripts/firewall-start`:
```
# Reinstate the ipset rules if they have been created already
[ "$(uname -m)" = "mips" ] && MATCH_SET='--set' || MATCH_SET='--match-set'
for ipSet in $(ipset -L | sed -n '/^Name:/s/^.* //p'); do
  case $ipSet in
    AcceptList) iptables-save | grep -q "$ipSet" || iptables -I INPUT -m set $MATCH_SET $ipSet src -j ACCEPT;;
    TorNodes|BlockedCountries|CustomBlock) iptables-save | grep -q "$ipSet" || iptables -I INPUT -m set $MATCH_SET $ipSet src -j DROP;;
    MicrosoftSpyServers) iptables-save | grep -q "$ipSet" || iptables -I FORWARD -m set $MATCH_SET $ipSet dst -j DROP;;
    *) iptables-save | grep -q "$ipSet" || iptables -I FORWARD -m set $MATCH_SET $ipSet src,dst -j DROP;;
  esac
done
```