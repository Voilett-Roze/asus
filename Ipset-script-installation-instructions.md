> This page is helpful instructions on installing the various maintained scripts if there are any issues please turn to the respective thread for support by the author.

# Tor and Countries Block 

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
For support on this script please visit this [forum thread](https://www.snbforums.com/threads/country-blocking-script.36732/) on SnBForums

# Malware-Filter

**Description**: This script checks security firms list over malware spreading ip addresses and blocks them both outgoing and incoming connections from contacting your network.

* Enable and format [JFFS](https://github.com/RMerl/asuswrt-merlin/wiki/JFFS) through WEB UI first,

* Then place [**this content**](https://gitlab.com/swe_toast/malware-filter/raw/master/malware-block) to `/jffs/scripts/malware-block`

* Then make it executable:
```
chmod +x /jffs/scripts//malware-block
```
* then append the following line to /jffs/scripts/services-start:
```
cru a malware-filter "0 */12 * * * /opt/bin/malware-block"
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
For support on this script please visit this [forum thread](https://www.snbforums.com/threads/malware-filter-bad-host-ipset.35423/) on SnBForums
