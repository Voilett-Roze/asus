___
> **NOTE:** This page is helpful instructions on installing the various maintained scripts if there are any issues please turn to the respective thread for support by the author.

>**ATTENTION:** Make sure that you have a backup of your [JFFS partition](https://github.com/RMerl/asuswrt-merlin/wiki/Jffs#backing-up-the-jffs-partition) in case something goes wrong. If your using multiple scripts from this make sure they dont run all at the same time that will decrease performance, use sleep command to make the script run at separate times or use && to make them run after each other.
___

### Table of Contents
* [MatchIP Utility - Search IPSet lists for an IP](#matchip-utility---search-ipset-lists-for-an-ip)  
* [Tor and Countries Block](#tor-and-countries-block)  
* [iblocklist-loader](#iblocklist-loader)  
* [Dynamically Ban Malicious IP's](#dynamically-ban-malicious-ips) 

# MatchIP Utility - Search IPset lists for an IP

When using the scripts from this page, there is a chance that a site/IP address you regularly access ends up being blocked. Depending on how many scripts you are using, it can be time consuming to try and determine which ipset list is causing the block. 

The below steps will show you how to add a command you can run that will search all of the ipset lists in use, irrespective of their script source, for a provided IP.

Follow the general script installation instructions:

* Enable and format [**JFFS**](https://github.com/RMerl/asuswrt-merlin/wiki/JFFS) through WEB UI first (if not already enabled)

* Using your favourite text editor (e.g. nano), add the below into **/jffs/configs/profile.add**

* Please make sure you only add the function that applies to the [**version of ipset**](https://github.com/RMerl/asuswrt-merlin/wiki/Using-ipset#ipset-version-and-router-models) that your router supports.

* For ipset v4:
```
MatchIP() { # Check IP against ipset lists
  if [ -z "$1" ]; then
    echo "Specify IP to check through ipset lists. Exiting."
  else
    GREEN='\033[0;32m'
    RED='\033[0;31m'
    NC='\033[0m' # No Color
    for TestList in $( (iptables -L -t raw && iptables -L) | grep " set" | tr -s ' ' | cut -d' ' -f7 | tr '\n' ' '); do
      ipset -q --test $TestList $1 && echo -e "$1 found in ${GREEN}${TestList}${NC}" || echo -e "$1 not found in ${RED}${TestList}${NC}"
    done
  fi
}
```
* For ipset v6:
```
MatchIP() { # Check IP against ipset lists
  if [ -z "$1" ]; then
    echo "Specify IP to check through ipset lists. Exiting."
  else
    GREEN='\033[0;32m'
    RED='\033[0;31m'
    NC='\033[0m' # No Color
    for TestList in $( (iptables -L -t raw && iptables -L) | grep "match-set" | tr -s ' ' | cut -d' ' -f7 | tr '\n' ' '); do
      ipset -q test $TestList $1 && echo -e "$1 found in ${GREEN}${TestList}${NC}" || echo -e "$1 not found in ${RED}${TestList}${NC}"
    done
  fi
}
```
* Restart your SSH session to pull in the profile change

* Use the command as below
```
MatchIP 8.8.8.8
```
```
# Sample output
8.8.8.8 not found in YAMalwareBlock1IP
8.8.8.8 not found in YAMalwareBlock2IP
8.8.8.8 not found in YAMalwareBlockCIDR
8.8.8.8 not found in WhitelistDomains
8.8.8.8 not found in BlacklistDomains
8.8.8.8 not found in I-BlocklistTheOnionRouterCIDR
8.8.8.8 not found in I-BlocklistTheOnionRouterSingle
8.8.8.8 not found in BluetackDshieldCIDR
8.8.8.8 not found in BluetackDshieldSingle
8.8.8.8 not found in BluetackSpiderCIDR
8.8.8.8 not found in BluetackSpiderSingle
```


# Tor and Countries Block 

This example shows how to block incoming connection from [Tor](https://www.torproject.org/) nodes (iphash set type — number of ip addresses) and how to block incoming connection from whole countries (nethash set type - number of ip subnets). 

* Enable and format [JFFS](https://github.com/RMerl/asuswrt-merlin/wiki/JFFS) through WEB UI first (if not already enabled)

* Then place [**this content**](https://raw.githubusercontent.com/shounak-de/misc-scripts/master/create-ipset-lists.sh) to `/jffs/scripts/create-ipset-lists.sh`

* Then make it executable:
```
chmod +x /jffs/scripts/create-ipset-lists.sh
```
* Finally call this at the end of your existing /jffs/scripts/services-start *or* /jffs/scripts/post-mount:
```
# Load ipset filter rules
sh /jffs/scripts/create-ipset-lists.sh
```

You may also run `/jffs/scripts/create-ipset-lists.sh` from command line or reboot router to apply new blocking rules immediately. Note that if you change the script country list or any other part, it would need a router reboot to take effect: _There are checks to prevent it from reloading the sets if the sets are already existing._

You can create a handy alias in your profile (in /opt/etc/profile or /jffs/configs/profile.add)

If you have ipset v4:
```
alias blockstats='iptables -vL | sed "2q;d"; (iptables -vL -t raw; iptables -vL) | grep -e " set"'
```
If you have ipset v6:
```
alias blockstats='iptables -vL | sed "2q;d"; (iptables -vL -t raw; iptables -vL) | grep "match-set"; ip6tables -vL | grep "match-set"'
```

Then you can just issue 'blockstats' from the command prompt to see how well your blocklists are doing (see blocked packet count and byte count)

Note that every time you do something on the web UI or through your [android app](https://play.google.com/store/apps/details?id=com.asus.aihome) or [ios app](https://appsto.re/gb/8hNN9.i) to control your router _that affects reloading the firewall rules_, `/jffs/scripts/firewall-start` will be called, so the iptables rules that are defined outside will be wiped out. To reinstate the rules as defined by this script, you'd need to add this to a script (say `/jffs/scripts/firewall-reinstate.sh`):
```
#!/bin/sh
# Reinstate the ipset rules if they have been created already
[ "$(uname -m)" = "mips" ] && MATCH_SET='--set' || MATCH_SET='--match-set'
for ipSet in $(ipset -L | sed -n '/^Name:/s/^.* //p'); do
  case $ipSet in
    AcceptList) iptables-save | grep -q "$ipSet" || iptables -I INPUT -m set $MATCH_SET $ipSet src -j ACCEPT;;
    WhitelistDomains) iptables-save | grep -q "$ipSet" || iptables -t raw -I PREROUTING -m set $MATCH_SET $ipSet src,dst -j ACCEPT;;
    TorNodes|BlockedCountries|CustomBlock) iptables-save | grep -q "$ipSet" || iptables -I INPUT -m set $MATCH_SET $ipSet src -j DROP;;
    MicrosoftSpyServers) iptables-save | grep -q "$ipSet" || iptables -I FORWARD -m set $MATCH_SET $ipSet dst -j DROP;;
    YAMalwareBlock*) iptables-save | grep -q "$ipSet" || iptables -t raw -I PREROUTING -m set $MATCH_SET $ipSet src -j DROP;;
    *) iptables-save | grep -q "$ipSet" || iptables -t raw -I PREROUTING -m set $MATCH_SET $ipSet src,dst -j DROP;;
  esac
done
```
And then call that from your _existing_ `/jffs/scripts/firewall-start`:
```
sh /jffs/scripts/firewall-reinstate.sh
```
For support on this script please visit this [forum thread](https://www.snbforums.com/threads/country-blocking-script.36732/page-2) on SnBForums
___

# iblocklist-loader

**Description**: This is a multi-purpose blocking script, and can be a replacement for the old [[Peerguardian|Legacy-Ipset-Scripts]] Scripts.

In addition, this script can also be used to block hackers, spiders, bogon ips, various organisations and ISPs, etc. This also blocks Tor and proxies as well. The full list of what it can block is available on the [iblocklist](https://www.iblocklist.com/lists) site. You can see the free lists on that site and their descriptions (by clicking on the list name).

The script uses the zipped IP range data from the iblocklist site and creates single ip sets and CIDR sets from that for ipset 6.x and uses iptreemap for ipset 4.x It also has both [WhitelistDomains](https://github.com/shounak-de/iblocklist-loader/blob/master/whitelist-domains.txt) and [BlacklistDomains](https://github.com/shounak-de/iblocklist-loader/blob/master/blacklist-domains.txt) functionality to explicitly handle any domains that you'd want to allow or block.

Follow the general script installation instructions:

* Enable and format [JFFS](https://github.com/RMerl/asuswrt-merlin/wiki/JFFS) through WEB UI first (if not already enabled)

* Then place [**this content**](https://raw.githubusercontent.com/shounak-de/iblocklist-loader/master/iblocklist-loader-v2.sh) to `/jffs/scripts/iblocklist-loader.sh`

After you decide what sources you'd like to block, these are the steps to use the script: Identify the lists in the script [in the top section](https://github.com/shounak-de/iblocklist-loader/blob/master/iblocklist-loader-v2.sh#L10-L79) and note the indexes (the number after the word `List`) You can than add your chosen indexes in the line that reads `BLOCKLIST_INDEXES=` 

* Then make it executable:
```
chmod +x /jffs/scripts/iblocklist-loader.sh
```

* Finally call this at the end of your existing /jffs/scripts/services-start *or* /jffs/scripts/post-mount:

```
# Load ipset filter rules
sh /jffs/scripts/iblocklist-loader.sh
```
There are other settings on the script that is documented within the script itself that should be self explanatory. If you have questions or need further details, please ask on the [forum thread](https://www.snbforums.com/threads/iblocklist-com-generic-ipset-loader-for-ipset-v6-and-v4.37976/) on SnBForums.
___