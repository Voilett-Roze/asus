___
> **NOTE:** This page is helpful instructions on installing the various maintained scripts if there are any issues please turn to the respective thread for support by the author.

>**ATTENTION:** Make sure that you have a backup of your [JFFS partition](https://github.com/RMerl/asuswrt-merlin/wiki/Jffs#backing-up-the-jffs-partition) incase something goes wrong.
___

# Tor and Countries Block 

This example shows how to block incoming connection from [Tor](https://www.torproject.org/) nodes (iphash set type — number of ip addresses) and how to block incoming connection from whole countries (nethash set type - number of ip subnets). 

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
alias blockstats='iptables -L -v | sed "2q;d"; iptables -L -v | grep " set"'
```
If you have ipset v6:
```
alias blockstats='iptables -L -v | sed "2q;d"; iptables -L -v | grep "match-set"; ip6tables -L -v | grep "match-set"'
```
Then you can just issue 'blockstats' from the command prompt to see how well your blocklists are doing (see blocked packet count and byte count)

Note that every time you do something on the web UI or through your [android app](https://play.google.com/store/apps/details?id=com.asus.aihome) or [ios app](https://appsto.re/gb/8hNN9.i) to control your router _that affects reloading the firewall rules_, `/jffs/scripts/firewall-start` will be called, so the iptables rules that are defined outside will be wiped out. To reinstate the rules as defined by this script, you'd need to add this to your _existing_ `/jffs/scripts/firewall-start`:
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
For support on this script please visit this [forum thread](https://www.snbforums.com/threads/country-blocking-script.36732/page-2) on SnBForums
___

# iblocklist-loader

**Description**: This is a perfect replacement for the old [Peerguardian Scripts](https://github.com/RMerl/asuswrt-merlin/wiki/Peerguardian-Scripts) it blocks various sources and has support for both ranges and single ip addresses and it has both [whitelist](https://github.com/shounak-de/iblocklist-loader/blob/master/whitelist-domains.txt) and [blacklist](https://github.com/shounak-de/iblocklist-loader/blob/master/blacklist-domains.txt) functionality.

In addition, this script can be used to block hackers, spiders, bogon ips, various organisations and ISPs, This also blocks Tor and proxies as well. The full list of what it can block is available on the [iblocklist](https://www.iblocklist.com/lists) site. You can see the free lists on that site and their descriptions (by clicking on the list name).

Follow the general script installation instructions:

* Enable and format [JFFS](https://github.com/RMerl/asuswrt-merlin/wiki/JFFS) through WEB UI first,

* Then place [**this content**](https://raw.githubusercontent.com/shounak-de/iblocklist-loader/master/iblocklist-loader-v2.sh) to `/jffs/scripts/iblocklist-loader.sh`

After you decide what sources you'd like to block, these are the steps to use the script: Identify the lists in the script [in the top section](https://github.com/shounak-de/iblocklist-loader/blob/master/iblocklist-loader-v2.sh#L10-L79) and note the indexes (the number after the word `List`) You can than add your chosen indexes in the line that reads `BLOCKLIST_INDEXES=` 

* Then make it executable:
```
chmod +x /jffs/scripts/iblocklist-loader.sh
```

* Finally call this at the end of your existing /jffs/firewall-start:

```
# Load ipset filter rules
sh /jffs/scripts/iblocklist-loader.sh
```
There are other settings on the script that is documented within the script itself that should be self explanatory. If you have questions or need further details, please ask on the [forum thread](https://www.snbforums.com/threads/iblocklist-com-generic-ipset-loader-for-ipset-v6-and-v4.37976/) on SnBForums
___
# Malware-Filter

**Description**: This script checks security firms list over malware spreading ip addresses and blocks them both outgoing and incoming connections from contacting your network.

* Enable and format [JFFS](https://github.com/RMerl/asuswrt-merlin/wiki/JFFS) through WEB UI first,

* Then place [**this content**](https://gitlab.com/swe_toast/malware-filter/raw/master/malware-block) to `/jffs/scripts/malware-block`

* Then make it executable:
```
chmod +x /jffs/scripts/malware-block
```
* then append the following line to /jffs/scripts/services-start:
```
cru a malware-filter "0 */12 * * * /jffs/scripts/malware-block"
```
This will make Malware-Filter run on a schedule it will run every 12th hour, to verify that the entry works after its added just type:

```
cru l
```
* Finally call this at the end of your existing /jffs/firewall-start:
```
# Load ipset filter rules
sh /jffs/scripts/malware-block
```
* To run this manually just type this command:
```
/jffs/scripts/malware-block
```
* Malware filter will also print to syslog so you dont have to check ssh to see if its working it should read something like this:
```
Apr  1 00:06:39 system: Malware-filter loaded 46822 unique ip addresses that will be rejected from contacting your router.
````

For support on this script please visit this [forum thread](https://www.snbforums.com/threads/malware-filter-bad-host-ipset.35423/) on SnBForums
___

# Privacy-Filter

**Description**: This script blocks [Telemetry](https://en.wikipedia.org/wiki/Criticism_of_Microsoft_Windows#Data_collection), [Shodan.io crawlers](https://www.shodan.io/) and an [Android Rootkit](https://www.kb.cert.org/vuls/id/624539), it supports both ipv4 and ipv6 out of the box however ipv6 blocking will only work on routers with [ipset version 6](https://github.com/RMerl/asuswrt-merlin/wiki/Using-ipset#ipset-version-and-router-models) installed.

* Enable and format [JFFS](https://github.com/RMerl/asuswrt-merlin/wiki/JFFS) through WEB UI first,

* Then place [**this content**](https://gitlab.com/swe_toast/privacy-filter/raw/master/privacy-filter) to `/jffs/scripts/privacy-filter`

* Then make it executable:
```
chmod +x /jffs/scripts/privacy-filter
```
* then append the following line to /jffs/scripts/services-start:
```
cru a privacy-filter "0 */12 * * * /jffs/scripts/privacy-filter"
```
This will make privacy-filter run on a schedule it will run every 12th hour, to verify that the entry works after its added just type:

```
cru l
```
* Finally call this at the end of your existing /jffs/firewall-start:
```
# Load ipset filter rules
sh /jffs/scripts/privacy-filter
```
* To run this manually just type this command:
```
/jffs/scripts/privacy-filter
```
* Privacy filter will also print to syslog so you dont have to check ssh to see if its working it should read something like this:
```
Apr  1 00:00:06 system: Privacy Filter (ipv4) loaded 190 unique ip addresses that will be rejected from contacting your  router.
````

For support on this script please visit this [forum thread](https://www.snbforums.com/threads/privacy-filter-another-ipset-script.36801/) on SnBForums
___