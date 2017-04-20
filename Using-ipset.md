
Since 3.0.0.4_270.26 [ipset](http://en.wikipedia.org/wiki/Netfilter#ipset) feature has been implemented. This is a Netfilter extension which should be able to:
* store multiple IP addresses and/or port numbers and match against a filter list using iptables
* dynamically update iptables rules against IP addresses or ports without a significant performance penalty
* express complex IP address and port based rulesets with one single iptables rule and benefit from the speed of IP sets.

> NOTE: For more documentation about the various commands using the ipset utility, please visit [this link](http://ipset.netfilter.org/)

## Ipset Version and Router Models

Newer router has [ipset version 6](http://ipset.netfilter.org/ipset.man.html) while older routers has ipset version 4 , ipset cant just be updated as a normal application it relies heavily on the kernel so please consult the chart below to see your ipset version.

| `Routers`    |`Ipset 4`|`Ipset 6`|
|--------------|:-------:|:-------:|
| `RT-N66U`    | x       |         |
| `RT-AC56U`   |         | x       |
| `RT-AC66U`   | x       |         |
| `RT-AC66U_B1`|         | x       |
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
> Note: The script with maintainers are linked in the chart to their respective installation instructions there on the wiki, within those instructions you will find information on how to install and where to get support. The Peerguardian scripts are considered legacy. Only use those if your router supports ipset version 4 and your capable to manage on your own, if you don't then consider using the iblocklist-loader instead it supports both ipset versions and have an active maintainer.

There is a full list of script that are maintained by users, most of the scripts are have various functions for blocking connections please read the description carefully before installing any of these scripts, not all scripts have maintainers and getting support on those scripts can be tricky.

>ATTENTION: Scripters, feel free to append to this list and then link installation instructions on the [installation instructions page](https://github.com/RMerl/asuswrt-merlin/wiki/Ipset-script-installation-instructions), please dont add full scripts to that page cause it gets messy "keep it light".

| `Scriptname` |`Ipset Version`|`Maintained by`|Description|
|--------------|:-------:|:-------------:|:----------------------------------:
|[Tor and Countries Block](https://github.com/RMerl/asuswrt-merlin/wiki/Ipset-script-installation-instructions#tor-and-countries-block)|4,6|redhat27|Blocks Tor nodes or countries| 
|[iblocklist-loader](https://github.com/RMerl/asuswrt-merlin/wiki/Ipset-script-installation-instructions#iblocklist-loader)|4,6|redhat27|Block or allow using any list from iblocklist| 
|[Malware Filter](https://github.com/RMerl/asuswrt-merlin/wiki/Ipset-script-installation-instructions#malware-filter)|4,6|swetoast|Blocks Malware Spreading ip addresses daily|
|[Privacy Filter](https://github.com/RMerl/asuswrt-merlin/wiki/Ipset-script-installation-instructions#privacy-filter)|4,6|swetoast|Blocks Telemetry, Trackers and Shodian.io|
|[Peerguardian v1](https://github.com/RMerl/asuswrt-merlin/wiki/Legacy-Ipset-Scripts#peer-guardian)|4| |Peerguardian|
|[Peerguardian v2](https://github.com/RMerl/asuswrt-merlin/wiki/Legacy-Ipset-Scripts#peer-guardian-v2)|4| |Peerguardian|
|[Peerguardian v3](https://github.com/RMerl/asuswrt-merlin/wiki/Legacy-Ipset-Scripts#peer-guardian-v3)|4| |Peerguardian|
|[Disable Windows 10 Tracking](https://github.com/RMerl/asuswrt-merlin/wiki/Legacy-Ipset-Scripts#disable-windows10-tracking)|4| |Blocks Telemetry|