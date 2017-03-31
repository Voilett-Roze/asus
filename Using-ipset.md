# Using ipset

Since 3.0.0.4_270.26 [ipset](http://en.wikipedia.org/wiki/Netfilter#ipset) feature has been implemented. This is a Netfilter extension which should be able to:
* store multiple IP addresses and/or port numbers and match against a filter list using iptables;
* dynamically update iptables rules against IP addresses or ports without a significant performance penalty
* express complex IP address and port based rulesets with one single iptables rule and benefit from the speed of IP sets.

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
|[Peerguardian v1](https://github.com/RMerl/asuswrt-merlin/wiki/Using-ipset#peer-guardian)|x| | |no|Peerguardian|
|[Peerguardian v2](https://github.com/RMerl/asuswrt-merlin/wiki/Using-ipset#peer-guardian-v2)|x| | |no|Peerguardian|
|[Peerguardian v3](https://github.com/RMerl/asuswrt-merlin/wiki/Using-ipset#peer-guardian-v3)|x| | |no|Peerguardian|

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
***

# Peer Guardian

> **NOTE:** _Peer Guardian scripts on this page supports only IPSET 4.x that will result in scripts not working on newer routers with IPSET 6.x. If you want to use IPSET 6.x for Peer Guardian and/or other block lists from [iblocklist.com](https://www.iblocklist.com/lists), you can check out [this](https://www.snbforums.com/threads/iblocklist-com-generic-ipset-loader-for-ipset-v6-and-v4.37976/) thread_

Supports only IPSET 4 (For IPSET 6, see [here](https://www.snbforums.com/threads/peer-guardian-rewrite-for-ipset-v6.37929/), the actual script [here](https://raw.githubusercontent.com/shounak-de/iblocklist-loader/master/iblocklist-loader.sh))

Another example is a [PeerGuardian](http://en.wikipedia.org/wiki/PeerGuardian) functionality right on router.

Please do not add this script to `/jffs/scripts/firewall-start` because it executes too long (~25 min on RT-N66U). Place following content to `/jffs/scripts/peerguardian.sh`
```
#!/bin/sh

# Loading ipset modules
lsmod | grep "ipt_set" > /dev/null 2>&1 || \
for module in ip_set ip_set_iptreemap ipt_set
do
    insmod $module
done

# Different routers got different iptables syntax
case $(uname -m) in
  armv7l)
    MATCH_SET='--match-set'
    ;;
  mips)
    MATCH_SET='--set'
    ;;
esac

# PeerGuardian rules
if [ "$(ipset --swap BluetackLevel1 BluetackLevel1 2>&1 | grep 'Unknown set')" != "" ]
then
    ipset --create BluetackLevel1 iptreemap
    [ -e /tmp/bluetack_lev1.lst ] || wget -q -O - "http://list.iblocklist.com/?list=bt_level1&fileformat=p2p&archiveformat=gz" | \
        gunzip | cut -d: -f2 | grep -E "^[-0-9.]+$" > /tmp/bluetack_lev1.lst
    for IP in $(cat /tmp/bluetack_lev1.lst)
    do
        ipset -A BluetackLevel1 $IP
    done
fi
iptables -I FORWARD -m set $MATCH_SET BluetackLevel1 src,dst -j DROP
```
and run it:
```
sh /jffs/scripts/peerguardian.sh
```
Please don't close SSH-session until it finishes. Script will blocks [over 8 000 000 IP's addresses](http://www.iblocklist.com/list.php?list=bt_level1) which anti-p2p activity has been seen from.

***
# Peer Guardian V2
Supports only IPSET 4 

Below is a speed optimized version of the peerguardian.sh script above. It does the same thing, but takes less than 30 seconds to run (the shortest run took 20 seconds on my RT-N66U). It might now be possible to run it from `/jffs/scripts/firewall-start`.

The script utilizes two sets: primary "BluetackLevel1" and temporary "BluetackLevel2". The IPs are bulk loaded into the temporary one and then swapped into the primary. Because of this approach it can also be run periodically on a running router to refresh the active set.
```
#!/bin/sh

# PeerGuardian rules

# Loading ipset modules
lsmod | grep "ipt_set" > /dev/null 2>&1 || \
for module in ip_set ip_set_iptreemap ipt_set; do
    insmod $module
done

# Different routers got different iptables syntax
case $(uname -m) in
  armv7l)
    MATCH_SET='--match-set'
    ;;
  mips)
    MATCH_SET='--set'
    ;;
esac

# Create the BluetackLevel1 (primary) if does not exists
if [ "$(ipset --swap BluetackLevel1 BluetackLevel1 2>&1 | grep 'Unknown set')" != "" ]; then
  ipset --create BluetackLevel1 iptreemap && \
  iptables -I FORWARD -m set $MATCH_SET BluetackLevel1 src,dst -j DROP
fi
# Destroy this transient set just in case
ipset --destroy BluetackLevel2 > /dev/null 2>&1
    
# Load the latest rule(s)

(echo -e "-N BluetackLevel2 iptreemap\n" && \
 nice wget -q -O - "http://list.iblocklist.com/?list=bt_level1&fileformat=p2p&archiveformat=gz" | \
    nice gunzip | nice cut -d: -f2 | nice grep -E "^[-0-9.]+$" | \
    nice sed 's/^/-A BluetackLevel2 /' && \
 echo -e "\nCOMMIT\n" \
) | \
nice ipset --restore && \
nice ipset --swap BluetackLevel2 BluetackLevel1 && \
nice ipset --destroy BluetackLevel2

exit $?
```
***
# Peer Guardian V3
Supports only IPSET 4 

If you want to have different blocklist, grouped by one, then this is a variant, where you can add multiple blocklists in one script.

```
#!/bin/sh

logger "PeerGuardian rules"

logger "Loading ipset modules"
lsmod | grep "ipt_set" > /dev/null 2>&1 || \
for module in ip_set ip_set_iptreemap ipt_set; do
    insmod $module
done

case $(uname -m) in
  armv7l)
    MATCH_SET='--match-set'
    ;;
  mips)
    MATCH_SET='--set'
    ;;
esac

logger "Create the BluetackLevel1 (primary) if does not exists"
if [ "$(ipset --swap BluetackLevel1 BluetackLevel1 2>&1 | grep 'Unknown set')" != "" ]; then
  ipset --create BluetackLevel1 iptreemap && \
  iptables -I FORWARD -m set $MATCH_SET BluetackLevel1 src,dst -j DROP
fi
logger "Destroy this transient set just in case"
ipset --destroy BluetackLevel2 > /dev/null 2>&1

logger "Load the latest rule(s)"

(
	 
	(
		(
		 nice wget -q -O - "http://list.iblocklist.com/?list=dgxtneitpuvgqqcpfulq&fileformat=p2p&archiveformat=gz" | \
	         nice gunzip | nice cut -d: -f2 | nice grep -E "^[-0-9.]+$"  \
		) && \
		(
	 	 nice wget -q -O - "http://list.iblocklist.com/?list=llvtlsjyoyiczbkjsxpf&fileformat=p2p&archiveformat=gz" | \
        	 nice gunzip | nice cut -d: -f2 | nice grep -E "^[-0-9.]+$"  \
		) && \
		(
		 nice wget -q -O - "http://list.iblocklist.com/?list=ydxerpxkpcfqjaybcssw&fileformat=p2p&archiveformat=gz" | \
		 nice gunzip | nice cut -d: -f2 | nice grep -E "^[-0-9.]+$"  \
		)
	) | \
	(  
      		nice sed '/^$/d' | \
	        nice sed 's/^/-A BluetackLevel2 /' | \
		nice sed '1s/^/-N BluetackLevel2 iptreemap\n/' && \
		echo -e "\nCOMMIT\n" \
	)
#) > output
) | \
nice ipset --restore && \
nice ipset --swap BluetackLevel2 BluetackLevel1 && \
nice ipset --destroy BluetackLevel2

logger "exiting Peerguarding rules"
exit $?
```
The output will be in router logs:
```
May 29 09:03:21 admin: PeerGuardian rules
May 29 09:03:21 admin: Loading ipset modules
May 29 09:03:21 admin: Create the BluetackLevel1 (primary) if does not exists
May 29 09:03:22 admin: Destroy this transient set just in case
May 29 09:03:22 admin: Load the latest rule(s)
May 29 09:04:04 admin: exiting Peerguarding rules
```
# Malware Filter
* Support Thread: https://www.snbforums.com/threads/malware-filter-bad-host-ipset.35423/
* Supports both IPSET 4 and 6

Grabs list of active ip addresses from abuse.ch and malwaredomainlist and blocks ips. 

its recommended not to store this script in firewall-start rather add the script to /jffs/scripts/malware-block 

then type this

> nano /jffs/scripts/services-start 

and append

> cru a malware-filter "0 */12 * * * /jffs/scripts/malware-block"

save it this will make malware-block and make it executable run every 12th hour and update the router, for all the temporary files it uses TMP dir for storage this will not cause wear and tear.
	
```shell
#!/bin/sh
# Author: Toast
# Contributers: Octopus, Tomsk, Neurophile, jimf, spalife, visortgw, Cedarhillguy, redhat27
# Testers: shooter40sw
# Supporters: lesandie
# Revision 22

blocklist=/jffs/malware-filter.list                     # Set your path here
fwoption=REJECT                                         # DROP/REJECT    (Default Value: REJECT)
retries=3                                               # Set number of tries here
regexp=`echo "\b([0-9]{1,3}\.){3}[0-9]{1,3}\b"`         # Dont change this value

case $(ipset -v | grep -o "v[4,6]") in
  v6)   MATCH_SET='--match-set'; CREATE='create'; ADD='add'; SWAP='swap'; IPHASH='hash:ip'; DESTROY='destroy'; LIST='list';
        lsmod | grep -q "xt_set" || \
        for module in ip_set ip_set_nethash ip_set_iphash xt_set
        do insmod $module; done ;;
  v4)   MATCH_SET='--set'; CREATE='--create'; ADD='--add'; SWAP='--swap'; IPHASH='iphash'; DESTROY='--destroy'; LIST='--list'; 
        lsmod | grep -q "ipt_set" || \
        for module in ip_set ip_set_nethash ip_set_iphash ipt_set
        do insmod $module; done ;;
  *)    logger -t system "Malware-filter detected an unsupported ipset version"; exit 1 ;;
esac

check_online () {
while ! ping -q -c 1 google.com >/dev/null 2>&1; do
    sleep 1
    WaitSeconds=$((WaitSeconds+1))
    [ $WaitSeconds -gt 300 ] && logger -t system "$0 Router not online! Aborting after a wait of 5 minutes..." && exit 1
done
}

get_list () {
url=https://gitlab.com/swe_toast/malware-filter/raw/master/malware-filter.list
if [ ! -f $blocklist ]; then wget $url -O $blocklist; get_source; else get_source; fi
}

get_source () {
wget -q --tries=$retries --show-progress -i $blocklist -O /tmp/malware-filter-raw.part
    awk '!/(^127\.)|(^10\.)|(^172\.1[6-9]\.)|(^172\.2[0-9]\.)|(^172\.3[0-1]\.)|(^192\.168\.)/' /tmp/malware-filter-raw.part > /tmp/malware-filter-presort.part
    cat /tmp/malware-filter-presort.part | grep -oE "$regexp" | sort -u > /tmp/malware-filter-sorted.part
}

run_ipset () {
echo "adding malware-filter rules to firewall this will take time."
! ipset $LIST malware-filter &>/dev/null
if [ $? -ne 0 ]
then    nice -n 15 ipset $CREATE malware-update $IPHASH
        if [ -f /opt/bin/xargs ]; then
        /opt/bin/xargs -P10 -I "PARAM" -n1 -a /tmp/malware-filter-sorted.part nice -n 15 ipset $ADD malware-update PARAM
        else cat /tmp/malware-filter-sorted.part | xargs -I {} nice -n 15 ipset $ADD malware-update {}; fi
        nice -n 15 ipset $SWAP malware-update malware-filter
        nice -n 15 ipset $DESTROY malware-update
else    nice -n 15 ipset $CREATE malware-filter $IPHASH
        if [ -f /opt/bin/xargs ]; then
        /opt/bin/xargs -P10 -I "PARAM" -n1 -a /tmp/malware-filter-sorted.part nice -n 15 ipset $ADD malware-filter PARAM
        else cat /tmp/malware-filter-sorted.part | xargs -I {} nice -n 15 ipset $ADD malware-filter {}; fi
fi }

set_firewall () {
for ipSet in $(ipset -L | sed -n '/^Name:/s/^.* //p'); do
    case $ipSet in
        malware-filter) iptables-save | grep -q "$ipSet" || iptables -I FORWARD -m set $MATCH_SET $ipSet src,dst -j $fwoption ;;
    esac
done
}

cleanup () {
logger -t system "Malware-filter loaded $(ipset -L malware-filter | wc -l | awk '{print $1-7}') unique ip addresses that will be rejected from contacting your router."
find /tmp -name 'malware-filter-*.part' -exec rm {} +
}

check_online
get_list
run_ipset
set_firewall
cleanup

exit $?
```
Save this list as malware-filter.list and set it in your relative path (see configuration part in script) you can also add more list by just appending to this list, if you don't know how to don't worry it will download a default list on its own.
```
https://ransomwaretracker.abuse.ch/downloads/RW_IPBL.txt
https://zeustracker.abuse.ch/blocklist.php?download=badips
https://feodotracker.abuse.ch/blocklist/?download=ipblocklist
http://www.malwaredomainlist.com/hostslist/ip.txt
https://rules.emergingthreats.net/fwrules/emerging-Block-IPs.txt
https://lists.blocklist.de/lists/ssh.txt
https://lists.blocklist.de/lists/bots.txt
```

# Privacy Filter
* Support thread: https://www.snbforums.com/threads/privacy-filter-another-ipset-script.36801/
* Supports both IPSET 4 and 6
* Optional: Entware (hostip) package

So this script tries to block [Telemetry](http://www.zdnet.com/article/windows-10-telemetry-secrets/) and some Chinese data collection centers for [Android rootkits](http://arstechnica.com/security/2016/11/powerful-backdoorrootkit-found-preinstalled-on-3-million-android-phones/) along with shodan.io scanners.

```shell
#!/bin/sh
# Author: Toast
# Contributers: Tomsk
# Supporters: lesandie
# Revision 17

blocklist=/jffs/privacy-filter.list                     # Set your path here 
retries=3                                               # Set number of tries here
fwoption=REJECT                                         # DROP/REJECT    (Default Value: REJECT)

# Dont change this value
regexp_v4=`echo "\b([0-9]{1,3}\.){3}[0-9]{1,3}\b"`
local_v4=`echo "!/(^127\.)|(^10\.)|(^172\.1[6-9]\.)|(^172\.2[0-9]\.)|(^172\.3[0-1]\.)|(^192\.168\.)/"`
regexp_v6=`echo "^(([0-9a-f]){1,4}:)+(:)?(([0-9a-f]){1,4}:)+(:)?(([0-9a-f]){1,4})"`
local_v6=`echo "!(^(fc00::)"`
# Dont change this value

case $(ipset -v | grep -o "v[4,6]") in
  v6)   MATCH_SET='--match-set'; CREATE='create'; ADD='add'; SWAP='swap'; IPHASH='hash:ip'; NETHASH='hash:net family inet'; FLUSH='flush'; DESTROY='destroy'; INET6='family inet6'; LIST='list'; 
        lsmod | grep -q "xt_set" || \
        for module in ip_set ip_set_nethash ip_set_iphash xt_set
        do insmod $module; done ;;
  v4)   MATCH_SET='--set'; CREATE='--create'; ADD='--add'; SWAP='--swap'; IPHASH='iphash'; NETHASH='nethash'; FLUSH='--flush'; DESTROY='--destroy' INET6=''; LIST='--list'; 
        lsmod | grep -q "ipt_set" || \
        for module in ip_set ip_set_nethash ip_set_iphash ipt_set
        do insmod $module; done ;;
  *)    logger -t system "Privacy-filter detected an unsupported ipset version"; exit 1 ;;
esac

check_online () {
while ! ping -q -c 1 google.com >/dev/null 2>&1; do
  sleep 1
  WaitSeconds=$((WaitSeconds+1))
  [ $WaitSeconds -gt 300 ] && logger -t system "$0: Warning: Router not online! Aborting after a wait of 5 minutes..." && exit 1
done
}

check_ipv6 () {
  ping6 -q -c 1 google.com >/dev/null 2>&1 && $1 || echo
}

get_list () {
url=https://gitlab.com/swe_toast/privacy-filter/raw/master/privacy-filter.list
if [ ! -f $blocklist ]; then wget -q --tries=$retries --show-progress $url -O $blocklist; fi }

fix_list () {
if [ -f $blocklist ]; then dos2unix $blocklist; fi }

run_ipv4_block () {
if [ -f /tmp/privacy-filter_ipv4_sorted.part ]; then rm /tmp/privacy-filter_ipv4_sorted.part; fi
    if [ -z "$(which hostip)" ]; then
        if [ -z "$(which /opt/bin/xargs)" ]
            then cat $blocklist | xargs -n 5 -I {} sh -c "traceroute -4 {} | head -1 >> "/tmp/privacy-filter_ipv4_raw.part""
            else cat $blocklist | /opt/bin/xargs -P 10 -n 5 -I {} sh -c "traceroute -4 {} | head -1 >> "/tmp/privacy-filter_ipv4_raw.part""; fi
                 cat /tmp/privacy-filter_ipv4_raw.part | grep -oE "$regexp_v4" >> /tmp/privacy-filter_ipv4_presort.part
else    if [ -z "$(which /opt/bin/xargs)" ]
            then cat $blocklist | xargs -n 5 -I {} sh -c "hostip {} >> "/tmp/privacy-filter_ipv4.prelist""
            else cat $blocklist | /opt/bin/xargs -P 10 -n 5 -I {} sh -c "hostip {} >> "/tmp/privacy-filter_ipv4.prelist""; fi
        fi
        
    if [ -f /tmp/privacy-filter_ipv4_presort.part ]; then
        awk $local_v4 /tmp/privacy-filter_ipv4_presort.part > /tmp/privacy-filter_ipv4.prelist; fi
        if [ -f /tmp/privacy-filter_ipv4.prelist ]; then sort -u /tmp/privacy-filter_ipv4.prelist > /tmp/privacy-filter_ipv4_sorted.part; fi
}
        
run_ipv6_block () {
if [ -f /tmp/privacy-filter_ipv6_sorted.part ]; then rm /tmp/privacy-filter_ipv6_sorted.part; fi
    if [ -z "$(which hostip)" ]; then
        if [ -z "$(which /opt/bin/xargs)" ]
            then cat $blocklist | xargs -n 5 -I {} sh -c "traceroute -6 {} | head -1 >> "/tmp/privacy-filter_ipv6_raw.part""
            else cat $blocklist | /opt/bin/xargs -P 10 -n 5 -I {} sh -c "traceroute -6 {} | head -1 >> "/tmp/privacy-filter_ipv6_raw.part""; fi
                 cat /tmp/privacy-filter_ipv6_raw.part | grep -oE "$regexp_v6" >> /tmp/privacy-filter_ipv6_presort.part
else    if [ -z "$(which /opt/bin/xargs)" ]
            then cat $blocklist | xargs -n 5 -I {} sh -c "hostip -6 {} >> "/tmp/privacy-filter_ipv6.prelist""
            else cat $blocklist | /opt/bin/xargs -P 10 -n 5 -I {} sh -c "hostip -6 {} >> "/tmp/privacy-filter_ipv6.prelist""; fi
        fi
        
    if [ -f /tmp/privacy-filter_ipv6_presort.part ]; then
        awk $local_v6 /tmp/privacy-filter_ipv6_presort.part > /tmp/privacy-filter_ipv6.prelist; fi
        if [ -f /tmp/privacy-filter_ipv6.prelist ]; then sort -u /tmp/privacy-filter_ipv6.prelist > /tmp/privacy-filter_ipv6_sorted.part; fi
}
        
run_ipset_4 () {
! ipset $LIST privacy-filter_ipv4 &>/dev/null 
if [ $? -ne 0 ]
then    nice -n 15 ipset $CREATE privacy-update_ipv4 $IPHASH
        cat /tmp/privacy-filter_ipv4_sorted.part | xargs -I {} ipset $ADD privacy-update_ipv4 {}
        nice -n 15 ipset $SWAP privacy-update_ipv4 privacy-filter_ipv4
        nice -n 15 ipset $DESTROY privacy-update_ipv4
else    nice -n 15 ipset $CREATE privacy-filter_ipv4 $IPHASH
        cat /tmp/privacy-filter_ipv4_sorted.part | xargs -I {} ipset $ADD privacy-filter_ipv4 {}
fi }

run_ipset_6 () {
! ipset $LIST privacy-filter_ipv6 &>/dev/null 
if [ $? -ne 0 ] 
then    nice -n 15 ipset $CREATE privacy-update_ipv6 $IPHASH $INET6
        cat /tmp/privacy-filter_ipv6_sorted.part | xargs -I {} ipset $ADD privacy-update_ipv6 {}
        nice -n 15 ipset $SWAP privacy-update_ipv6 privacy-filter_ipv6
        nice -n 15 ipset $DESTROY privacy-update_ipv6
else    nice -n 15 ipset $CREATE privacy-filter_ipv6 $IPHASH $INET6
        cat /tmp/privacy-filter_ipv6_sorted.part | xargs -I {} ipset $ADD privacy-filter_ipv6 {}
fi }

run_blocklists () {
run_ipv4_block
case $(ipset -v | grep -oE "ipset v[0-9]") in
*v6) check_ipv6 run_ipv6_block ;;
esac }

run_ipset () {
run_ipset_4
case $(ipset -v | grep -oE "ipset v[0-9]") in
*v6) check_ipv6 run_ipset_6 ;;
esac }

set_firewall () {
for ipSet in $(ipset -L | sed -n '/^Name:/s/^.* //p'); do
    case $ipSet in
        privacy-filter_ipv4) iptables-save | grep -q "$ipSet" || iptables -I FORWARD -m set $MATCH_SET $ipSet src,dst -j $fwoption ;;
        privacy-filter_ipv6) iptables-save | grep -q "$ipSet" || ip6tables -I FORWARD -m set $MATCH_SET $ipSet src,dst -j $fwoption ;;
    esac
done 
}

logipv6 () {
logger -s -t system "Privacy Filter (ipv6) loaded $(ipset -L  privacy-filter_ipv6 | wc -l | awk '{print $1-7}') unique ip addresses that will be rejected from contacting your router."
}

cleanup () {
find /tmp -name 'privacy-filter_ipv*.part' -exec rm {} +
logger -s -t system "Privacy Filter (ipv4) loaded $(ipset -L  privacy-filter_ipv4 | wc -l | awk '{print $1-7}') unique ip addresses that will be rejected from contacting your router."
check_ipv6 logipv6
}

check_online
fix_list
run_blocklists
run_ipset
set_firewall
cleanup

exit $?
```
Note: save this list as [privacy-filter.list](https://gitlab.com/swe_toast/privacy-filter/raw/master/privacy-filter.list) in your path on the router, if you set this file in the wrong place the script will automatically download a new copy and set it at either path or failover path.

