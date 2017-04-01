> NOTE: For more documentation about the various commands using the ipset utility, please visit [this link](http://ipset.netfilter.org/)


Since 3.0.0.4_270.26 [ipset](http://en.wikipedia.org/wiki/Netfilter#ipset) feature has been implemented. This is a Netfilter extension which should be able to:
* store multiple IP addresses and/or port numbers and match against a filter list using iptables
* dynamically update iptables rules against IP addresses or ports without a significant performance penalty
* express complex IP address and port based rulesets with one single iptables rule and benefit from the speed of IP sets.

## Ipset Version and Router Models

Newer router has [ipset version 6](http://ipset.netfilter.org/ipset.man.html) while older routers has ipset version 4 , ipset cant just be updated as a normal application it relies heavily on the kernel so please consult the chart below to see your ipset version.

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

## Ipset Scripts 

There is a full list of script that are maintained by users, most of the scripts are have various functions for blocking connections please read the description carefully before installing any of these scripts, not all scripts have maintainers and getting support on those scripts can be tricky.

> Note: The script with maintainers are linked in the chart to their respective installation instructions there on the wiki, within those instructions you will find information on how to install and where to get support. The Peerguardian scripts are considered legacy. Only use those if your router supports ipset version 4 and your capable to manage on your own, if you don't then consider using the iblocklist-loader instead it supports both ipset versions and have an active maintainer.


| `Scriptname` |`Ipset 4`|`Ipset 6`|`Maintained by`|`Supports other platforms`|Description|
|--------------|:-------:|:-------:|:-------------:|:------------------------:|:----------:
|[Tor and Countries Block](https://github.com/RMerl/asuswrt-merlin/wiki/Ipset-script-installation-instructions)|x|x|redhat27|no|Blocks Tor nodes or countries| 
|[iblocklist-loader](https://github.com/RMerl/asuswrt-merlin/wiki/Ipset-script-installation-instructions#iblocklist-loader)|x|x|redhat27|yes|Block or allow using any list from iblocklist| 
|[Malware Filter](https://github.com/RMerl/asuswrt-merlin/wiki/Ipset-script-installation-instructions#malware-filter)|x|x|swetoast|yes|Blocks Malware Spreading ip addresses daily|
|[Privacy Filter](https://github.com/RMerl/asuswrt-merlin/wiki/Ipset-script-installation-instructions#privacy-filter)|x|x|swetoast|yes|Blocks Telemetry, Trackers and Shodian.io|
|[Peerguardian v1](https://github.com/RMerl/asuswrt-merlin/wiki/Peerguardian-Scripts#peer-guardian)|x| | |no|Peerguardian|
|[Peerguardian v2](https://github.com/RMerl/asuswrt-merlin/wiki/Peerguardian-Scripts#peer-guardian-v2)|x| | |no|Peerguardian|
|[Peerguardian v3](https://github.com/RMerl/asuswrt-merlin/wiki/Peerguardian-Scripts#peer-guardian-v3)|x| | |no|Peerguardian|
