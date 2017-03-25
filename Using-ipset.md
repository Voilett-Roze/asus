# Using ipset

Since 3.0.0.4_270.26 [ipset](http://en.wikipedia.org/wiki/Netfilter#ipset) feature has been implemented. This is a Netfilter extension which should be able to:
* store multiple IP addresses and/or port numbers and match against a filter list using iptables;
* dynamically update iptables rules against IP addresses or ports without a significant performance penalty;
* express complex IP address and port based rulesets with one single iptables rule and benefit from the speed of IP sets.

> **NOTE:** _Peer Guardian scripts on this page supports only IPSET 4.x that will result in scripts not working on newer routers with IPSET 6.x. If you want to use IPSET 6.x for Peer Guardian and/or other block lists from [iblocklist.com](https://www.iblocklist.com/lists), you can check out [this](https://www.snbforums.com/threads/iblocklist-com-generic-ipset-loader-for-ipset-v6-and-v4.37976/) thread_

# Tor, Countries, M$ Telemetry, BruteForceLogins Block 
Supports both IPSET 4 and 6

This is an example of using [ipset utility](http://manpages.ubuntu.com/manpages/lucid/man8/ipset.8.html) with two different set types: iphash and nethash. The example shows how to block incoming connection from [Tor](https://www.torproject.org/) nodes (iphash set type — number of ip addresses) and how to block incoming connection from whole countries (nethash set type - number of ip subnets). 

Please, enable and format [JFFS](https://github.com/RMerl/asuswrt-merlin/wiki/JFFS) through WEB UI, place this content to `/jffs/scripts/create-ipset-lists.sh`

```
#!/bin/sh

# snbforums thread:
# https://www.snbforums.com/threads/country-blocking-script.36732/page-2#post-311407

# Re-download blocklist if locally saved blocklist is older than this many days
BLOCKLISTS_SAVE_DAYS=15

# For the users of mips routers (kernel 2.x): You can now block sources with IPv6 with country blocklists
# Enable if you want to add huge country IPv6 netmask lists directly into ip6tables rules.
# Also, enabling this will add a *lot* of processing time!
# Note: This has no effect *if* you have ipset v6: It will always use ipset v6 for IPv6 country blocklists regardless of whether this is enabled or not.
USE_IP6TABLES_IF_IPSETV6_UNAVAILABLE=disabled # [enabled|disabled]

# Block incoming traffic from some countries. cn and pk is for China and Pakistan. See other countries code at http://www.ipdeny.com/ipblocks/
BLOCKED_COUNTRY_LIST="br cn kp kr pk sa tr tw ua vn"

# Use DROP or REJECT for iptable rule for the ipset. Briefly, for DROP, attacker (or IP being blocked) will get no response and timeout, and REJECT will send immediate response of destination-unreachable (Attacker will know your IP is actively rejecting requests)
# See: http://www.chiark.greenend.org.uk/~peterb/network/drop-vs-reject and http://serverfault.com/questions/157375/reject-vs-drop-when-using-iptables
IPTABLES_RULE_TARGET=DROP # [DROP|REJECT]

# Preparing folder to cache downloaded files
IPSET_LISTS_DIR=/jffs/ipset_lists
[ -d "$IPSET_LISTS_DIR" ] || mkdir -p $IPSET_LISTS_DIR

# Different routers got different iptables and ipset syntax
case $(ipset -v | grep -o "v[4,6]") in
  v6)
    MATCH_SET='--match-set'; CREATE='create'; ADD='add'; SWAP='swap'; IPHASH='hash:ip'; NETHASH='hash:net family inet'; NETHASH6='hash:net family inet6'; SETNOTFOUND='name does not exist'
    # Loading ipset modules
    lsmod | grep -q "xt_set" || \
    for module in ip_set ip_set_nethash ip_set_iphash xt_set; do
      insmod $module
    done;;
  v4)
    MATCH_SET='--set'; CREATE='--create'; ADD='--add'; SWAP='--swap'; IPHASH='iphash'; NETHASH='nethash'; SETNOTFOUND='Unknown set'
    # Loading ipset modules
    lsmod | grep -q "ipt_set" || \
    for module in ip_set ip_set_nethash ip_set_iphash ipt_set; do
      insmod $module
    done;;
  *)
    logger -t Firewall "$0: Unknown ipset version: $(ipset -v). Exiting."
    exit 1;;
esac

# Wait if this is run early on (before the router has internet connectivity) [Needed by wget to download files]
while ! ping -q -c 1 google.com &>/dev/null; do
  sleep 1
  WaitSeconds=$((WaitSeconds+1))
  [ $WaitSeconds -gt 300 ] && logger -t Firewall "$0: Warning: Router not online! Aborting after a wait of 5 minutes..." && exit 1
done

# Allow traffic from AcceptList [IPv4 only] [$IPSET_LISTS_DIR/whitelist.lst can contain a combination of IPv4 IP or IPv4 netmask]
if [ -e $IPSET_LISTS_DIR/whitelist.lst ]; then
  if $(ipset $SWAP AcceptList AcceptList 2>&1 | grep -q "$SETNOTFOUND"); then
    ipset $CREATE AcceptList $NETHASH
    [ $? -eq 0 ] && entryCount=0
    for IP in $(cat $IPSET_LISTS_DIR/whitelist.lst); do
      [ "${IP##*/}" == "$IP" ] && ipset $ADD AcceptList $IP/31 || ipset $ADD AcceptList $IP
      [ $? -eq 0 ] && entryCount=$((entryCount+1))
    done
    logger -t Firewall "$0: Added AcceptList ($entryCount entries)"
  fi
  iptables-save | grep -q AcceptList || iptables -I INPUT -m set $MATCH_SET AcceptList src -j ACCEPT
fi

# Block traffic from known brute-force ssh login attempters [IPv4 nodes only]
if $(ipset $SWAP BruteForceLogins BruteForceLogins 2>&1 | grep -q "$SETNOTFOUND"); then
  ipset $CREATE BruteForceLogins $IPHASH
  [ $? -eq 0 ] && entryCount=0
  [ ! -e "$IPSET_LISTS_DIR/brute.lst" -o -n "$(find $IPSET_LISTS_DIR/brute.lst -mtime +$BLOCKLISTS_SAVE_DAYS -print 2>/dev/null)" ] && wget -q -O $IPSET_LISTS_DIR/brute.lst https://lists.blocklist.de/lists/bruteforcelogin.txt
  for IP in $(cat $IPSET_LISTS_DIR/brute.lst); do
    ipset $ADD BruteForceLogins $IP
    [ $? -eq 0 ] && entryCount=$((entryCount+1))
  done
  logger -t Firewall "$0: Added BruteForceLogins list ($entryCount entries)"
fi
iptables-save | grep -q BruteForceLogins || iptables -I INPUT -m set $MATCH_SET BruteForceLogins src -j $IPTABLES_RULE_TARGET

# Block traffic from Tor nodes [IPv4 nodes only]
if $(ipset $SWAP TorNodes TorNodes 2>&1 | grep -q "$SETNOTFOUND"); then
  ipset $CREATE TorNodes $IPHASH
  [ $? -eq 0 ] && entryCount=0
  [ ! -e "$IPSET_LISTS_DIR/tor.lst" -o -n "$(find $IPSET_LISTS_DIR/tor.lst -mtime +$BLOCKLISTS_SAVE_DAYS -print 2>/dev/null)" ] && wget -q -O $IPSET_LISTS_DIR/tor.lst http://torstatus.blutmagie.de/ip_list_all.php/Tor_ip_list_ALL.csv
  for IP in $(cat $IPSET_LISTS_DIR/tor.lst); do
    ipset $ADD TorNodes $IP
    [ $? -eq 0 ] && entryCount=$((entryCount+1))
  done
  logger -t Firewall "$0: Added TorNodes list ($entryCount entries)"
fi
iptables-save | grep -q TorNodes || iptables -I INPUT -m set $MATCH_SET TorNodes src -j $IPTABLES_RULE_TARGET

# Country blocking by nethashes [Both IPv4 and IPv6 sources]
if $(ipset $SWAP BlockedCountries BlockedCountries 2>&1 | grep -q "$SETNOTFOUND"); then
  ipset $CREATE BlockedCountries $NETHASH
  for country in ${BLOCKED_COUNTRY_LIST}; do
    entryCount=0
    [ ! -e "$IPSET_LISTS_DIR/$country.lst" -o -n "$(find $IPSET_LISTS_DIR/$country.lst -mtime +$BLOCKLISTS_SAVE_DAYS -print 2>/dev/null)" ] && wget -q -O $IPSET_LISTS_DIR/$country.lst http://www.ipdeny.com/ipblocks/data/aggregated/$country-aggregated.zone
    for IP in $(cat $IPSET_LISTS_DIR/$country.lst); do
      ipset $ADD BlockedCountries $IP
      [ $? -eq 0 ] && entryCount=$((entryCount+1))
    done
    logger -t Firewall "$0: Added country [$country] to BlockedCountries list ($entryCount entries)"
  done
fi
iptables-save | grep -q BlockedCountries || iptables -I INPUT -m set $MATCH_SET BlockedCountries src -j $IPTABLES_RULE_TARGET
if [ $(nvram get ipv6_fw_enable) -eq 1 -a "$(nvram get ipv6_service)" != "disabled" ]; then
  if $(ipset $SWAP BlockedCountries6 BlockedCountries6 2>&1 | grep -q "$SETNOTFOUND"); then
    [  -n "$NETHASH6" ] && ipset $CREATE BlockedCountries6 $NETHASH6
    for country in ${BLOCKED_COUNTRY_LIST}; do
      [ -e "/tmp/ipv6_country_blocks_loaded" ] && logger -t Firewall "$0: Country block rules has already been loaded into ip6tables... Skipping." && break
      entryCount=0
      [ ! -e "$IPSET_LISTS_DIR/${country}6.lst" -o -n "$(find $IPSET_LISTS_DIR/${country}6.lst -mtime +$BLOCKLISTS_SAVE_DAYS -print 2>/dev/null)" ] && wget -q -O $IPSET_LISTS_DIR/${country}6.lst http://www.ipdeny.com/ipv6/ipaddresses/aggregated/${country}-aggregated.zone
      for IP6 in $(cat $IPSET_LISTS_DIR/${country}6.lst); do
        if [ -n "$NETHASH6" ]; then
          ipset $ADD BlockedCountries6 $IP6
        elif [ $USE_IP6TABLES_IF_IPSETV6_UNAVAILABLE = "enabled" ]; then
          ip6tables -A INPUT -s $IP6 -j $IPTABLES_RULE_TARGET
        fi
        [ $? -eq 0 ] && entryCount=$((entryCount+1))
      done
      if [ -n "$NETHASH6" ]; then
        logger -t Firewall "$0: Added country [$country] to BlockedCountries6 list ($entryCount entries)"
      elif [ $USE_IP6TABLES_IF_IPSETV6_UNAVAILABLE = "enabled" ]; then
        logger -t Firewall "$0: Added country [$country] to ip6tables rules ($entryCount entries)"
      fi
    done
  fi
  if [ -n "$NETHASH6" ]; then
    ip6tables -L | grep -q BlockedCountries6 || ip6tables -I INPUT -m set $MATCH_SET BlockedCountries6 src -j $IPTABLES_RULE_TARGET
  elif [ $USE_IP6TABLES_IF_IPSETV6_UNAVAILABLE = "enabled" -a ! -e "/tmp/ipv6_country_blocks_loaded" ]; then
    logger -t Firewall "$0: Creating [/tmp/ipv6_country_blocks_loaded] to prevent accidental reloading of country blocklists in ip6table rules."
    touch /tmp/ipv6_country_blocks_loaded
  fi
fi

# Block Microsoft telemetry spying servers [IPv4 only]
if $(ipset $SWAP MicrosoftSpyServers MicrosoftSpyServers 2>&1 | grep -q "$SETNOTFOUND"); then
  ipset $CREATE MicrosoftSpyServers $IPHASH
  [ $? -eq 0 ] && entryCount=0
  for IP in 23.99.10.11 63.85.36.35 63.85.36.50 64.4.6.100 64.4.54.22 64.4.54.32 64.4.54.254 \
        65.52.100.7 65.52.100.9 65.52.100.11 65.52.100.91 65.52.100.92 65.52.100.93 65.52.100.94 \
        65.55.29.238 65.55.39.10 65.55.44.108 65.55.163.222 65.55.252.43 65.55.252.63 65.55.252.71 \
        65.55.252.92 65.55.252.93 66.119.144.157 93.184.215.200 104.76.146.123 111.221.29.177 \
        131.107.113.238 131.253.40.37 134.170.52.151 134.170.58.190 134.170.115.60 134.170.115.62 \
        134.170.188.248 157.55.129.21 157.55.133.204 157.56.91.77 168.62.187.13 191.234.72.183 \
        191.234.72.186 191.234.72.188 191.234.72.190 204.79.197.200 207.46.223.94 207.68.166.254; do
    ipset $ADD MicrosoftSpyServers $IP
    [ $? -eq 0 ] && entryCount=$((entryCount+1))
  done
  logger -t Firewall "$0: Added MicrosoftSpyServers list ($entryCount entries)"
fi
iptables-save | grep -q MicrosoftSpyServers || iptables -I FORWARD -m set $MATCH_SET MicrosoftSpyServers dst -j $IPTABLES_RULE_TARGET

# Block traffic from custom block list [IPv4 only]
if [ -e $IPSET_LISTS_DIR/custom.lst ]; then
  if $(ipset $SWAP CustomBlock CustomBlock 2>&1 | grep -q "$SETNOTFOUND"); then
    ipset $CREATE CustomBlock $IPHASH
    [ $? -eq 0 ] && entryCount=0
    for IP in $(cat $IPSET_LISTS_DIR/custom.lst); do
      ipset $ADD CustomBlock $IP
      [ $? -eq 0 ] && entryCount=$((entryCount+1))
    done
    logger -t Firewall "$0: Added CustomBlock list ($entryCount entries)"
  fi
  iptables-save | grep -q CustomBlock || iptables -I INPUT -m set $MATCH_SET CustomBlock src -j $IPTABLES_RULE_TARGET
fi
```

then make it executable:
```
chmod +x /jffs/scripts/create-ipset-lists.sh
```
and then call this at the end of your existing /jffs/firewall-start:
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

Note that every time you do something on the web UI or through your [android app] (https://play.google.com/store/apps/details?id=com.asus.aihome) to control your router _that affects reloading the firewall rules_, `/jffs/scripts/firewall-start` will be called, so the iptables rules that are defined outside will be wiped out. To reinstate the rules as defined by this script, you'd need to add this to your _existing_ `/jffs/scripts/firewall-start`:
```
# Reinstate the ipset rules if they have been created already
[ "$(uname -m)" = "mips" ] && MATCH_SET='--set' || MATCH_SET='--match-set'
for ipSet in $(ipset -L | sed -n '/^Name:/s/^.* //p'); do
  case $ipSet in
    AcceptList) iptables-save | grep -q "$ipSet" || iptables -I INPUT -m set $MATCH_SET $ipSet src -j ACCEPT;;
    BruteForceLogins|TorNodes|BlockedCountries|CustomBlock) iptables-save | grep -q "$ipSet" || iptables -I INPUT -m set $MATCH_SET $ipSet src -j DROP;;
    MicrosoftSpyServers) iptables-save | grep -q "$ipSet" || iptables -I FORWARD -m set $MATCH_SET $ipSet dst -j DROP;;
    *) iptables-save | grep -q "$ipSet" || iptables -I FORWARD -m set $MATCH_SET $ipSet src,dst -j DROP;;
  esac
done
```
***
## Peer Guardian
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
## Peer Guardian V2
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

##Peerguardian V3
Supports only IPSET 4 

If you want to have different blocklist, grouped by one, then this is a variant, where you can add multiple blocklists in one script...

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
## Malware Filter
* Support Thread: https://www.snbforums.com/threads/malware-filter-bad-host-ipset.35423/
* Supports both IPSET 4 and 6

Grabs list of active ip addresses from abuse.ch and malwaredomainlist and blocks ips. 

its recommended not to store this script in firewall-start rather add the script to /jffs/scripts/malware-block 

then type this

> nano /jffs/scripts/services-start 

and append

> cru a malware-filter "0 */12 * * * /jffs/scripts/malware-block"

save it this will make malware-block run every 12th hour and update the router, for all the temporary files it uses TMP dir for storage this will not cause wear and tear.

```
#!/bin/sh
# Author: Toast
# Contributers: Octopus, Tomsk, Neurophile, jimf, spalife, visortgw, Cedarhillguy, redhat27
# Testers: shooter40sw
# Supporters: lesandie
# Revision 19

blocklist=/jffs/malware-filter.list                     # Set your path here
retries=3                                               # Set number of tries here
regexp=`echo "\b([0-9]{1,3}\.){3}[0-9]{1,3}\b"`         # Dont change this value

case $(ipset -v | grep -o "v[4,6]") in
  v6)
    MATCH_SET='--match-set'; CREATE='create'; ADD='add'; SWAP='swap'; IPHASH='hash:ip'; NETHASH='hash:net family inet'; FLUSH='flush'; DESTROY='destroy';
    lsmod | grep -q "xt_set" || \
    for module in ip_set ip_set_nethash ip_set_iphash xt_set; do
      insmod $module
    done;;
  v4)
    MATCH_SET='--set'; CREATE='--create'; ADD='--add'; SWAP='--swap'; IPHASH='iphash'; NETHASH='nethash'; FLUSH='--flush'; DESTROY='--destroy'
    lsmod | grep -q "ipt_set" || \
    for module in ip_set ip_set_nethash ip_set_iphash ipt_set; do
      insmod $module
    done;;
  *) echo "unsupported version"; exit 1 ;;
esac

check_online () {
  ping -q -c 1 google.com >/dev/null 2>&1 && get_list || exit 1
}

get_list () {
url=https://gitlab.com/swe_toast/malware-filter/raw/master/malware-filter.list
if [ ! -f $blocklist ]
then wget $url -O $blocklist; get_source; else get_source; fi 
}

get_source () {
wget -q --tries=$retries --show-progress -i $blocklist -O /tmp/malware-filter-raw.part
    awk '!/(^127\.)|(^10\.)|(^172\.1[6-9]\.)|(^172\.2[0-9]\.)|(^172\.3[0-1]\.)|(^192\.168\.)/' /tmp/malware-filter-raw.part > /tmp/malware-filter-presort.part
    cat /tmp/malware-filter-presort.part | grep -oE "$regexp" | sort -u > /tmp/malware-filter-sorted.part
}

run_ipset () {
echo "adding ipset rule to firewall this will take time."
ipset -L malware-filter >/dev/null 2>&1
if [ $? -ne 0 ]; then
    if [ "$(ipset --swap malware-filter malware-filter 2>&1 | grep -E 'Unknown set|The set with the given name does not exist')" != "" ]; then
    nice -n 2 ipset $CREATE malware-filter $IPHASH 
    if [ -f /opt/bin/xargs ]; then
    /opt/bin/xargs -P10 -I "PARAM" -n1 -a /tmp/malware-filter-sorted.part nice -n 2 ipset  $ADD malware-filter PARAM
    else cat /tmp/malware-filter-sorted.part | xargs -I {} ipset $ADD malware-filter {}; fi
fi
else
    nice -n 2 ipset $CREATE malware-update $IPHASH 
    if [ -f /opt/bin/xargs ]; then
    /opt/bin/xargs -P10 -I "PARAM" -n1 -a /tmp/malware-filter-sorted.part nice -n 2 ipset  $ADD malware-update PARAM
    else cat /tmp/malware-filter-sorted.part | xargs -I {} ipset $ADD malware-update {}; fi
    nice -n 2 ipset $SWAP malware-update malware-filter
    nice -n 2 ipset $DESTROY malware-update
fi
iptables -L | grep malware-filter > /dev/null 2>&1
if [ $? -ne 0 ]; then
    nice -n 2 iptables -I FORWARD -m set $MATCH_SET malware-filter src,dst -j REJECT
else
    nice -n 2 iptables -D FORWARD -m set $MATCH_SET malware-filter src,dst -j REJECT
    nice -n 2 iptables -I FORWARD -m set $MATCH_SET malware-filter src,dst -j REJECT
fi }

cleanup () {
logger -s -t system "Malware Filter loaded $(ipset -L malware-filter | wc -l | awk '{print $1-7}') unique ip addresses."
find /tmp -name 'malware-filter-*.part' -exec rm {} +
}

check_online
run_ipset
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

## Privacy Filter
* Support thread: https://www.snbforums.com/threads/privacy-filter-another-ipset-script.36801/
* Supports both IPSET 4 and 6
* Optional: Entware (hostip) package

So this script tries to block [Telemetry](http://www.zdnet.com/article/windows-10-telemetry-secrets/) and some Chinese data collection centers for [Android rootkits](http://arstechnica.com/security/2016/11/powerful-backdoorrootkit-found-preinstalled-on-3-million-android-phones/) along with shodan.io scanners.

```
#!/bin/sh
# Author: Toast
# Contributers: Tomsk
# Supporters: lesandie
# Revision 15

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
  v6)
    MATCH_SET='--match-set'; CREATE='create'; ADD='add'; SWAP='swap'; IPHASH='hash:ip'; NETHASH='hash:net family inet'; FLUSH='flush'; DESTROY='destroy'; INET6='family inet6';
    lsmod | grep -q "xt_set" || \
    for module in ip_set ip_set_nethash ip_set_iphash xt_set; do
      insmod $module
    done;;
  v4)
    MATCH_SET='--set'; CREATE='--create'; ADD='--add'; SWAP='--swap'; IPHASH='iphash'; NETHASH='nethash'; FLUSH='--flush'; DESTROY='--destroy' INET6=''
    lsmod | grep -q "ipt_set" || \
    for module in ip_set ip_set_nethash ip_set_iphash ipt_set; do
      insmod $module
    done;;
  *) echo "unsupported version"; exit 1 ;;
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
if [ ! -f $blocklist ]
then wget -q --tries=$retries --show-progress $url -O $blocklist; fi }

fix_list () {
if [ -f $blocklist ]
then dos2unix $blocklist; fi
}

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
ipset -L privacy-filter_ipv4 >/dev/null 2>&1
if [ $? -ne 0 ]; then
   if [ "$(ipset $SWAP privacy-filter_ipv4 privacy-filter_ipv4 2>&1 | grep -E 'Unknown set|The set with the given name does not exist')" != "" ]; then
   nice ipset $CREATE privacy-filter_ipv4 $IPHASH
   cat /tmp/privacy-filter_ipv4_sorted.part | xargs -I {} ipset $ADD privacy-filter_ipv4 {}
fi
else
   nice -n 2 ipset $CREATE privacy-update_ipv4 $IPHASH
   cat /tmp/privacy-filter_ipv4_sorted.part | xargs -I {} ipset $ADD privacy-update_ipv4 {}
   nice -n 2 ipset $SWAP privacy-update_ipv4 privacy-filter_ipv4
   nice -n 2 ipset $DESTROY privacy-update_ipv4
fi
iptables -L | grep privacy-filter_ipv4 > /dev/null 2>&1
if [ $? -ne 0 ]; then
   nice -n 2 iptables -I FORWARD -m set $MATCH_SET privacy-filter_ipv4 src,dst -j $fwoption
else
   nice -n 2 iptables -D FORWARD -m set $MATCH_SET privacy-filter_ipv4 src,dst -j $fwoption
   nice -n 2 iptables -I FORWARD -m set $MATCH_SET privacy-filter_ipv4 src,dst -j $fwoption
fi }

run_ipset_6 () {
ipset -L privacy-filter_ipv6 >/dev/null 2>&1
if [ $? -ne 0 ]; then
   if [ "$(ipset $SWAP privacy-filter_ipv6 privacy-filter_ipv6 2>&1 | grep -E 'Unknown set|The set with the given name does not exist')" != "" ]; then
   nice ipset $CREATE privacy-filter_ipv6 $IPHASH $INET6
   cat /tmp/privacy-filter_ipv6_sorted.part | xargs -I {} ipset $ADD privacy-filter_ipv6 {}
fi
else
   nice -n 2 ipset -N privacy-update_ipv6 $IPHASH $INET6
   cat /tmp/privacy-filter_ipv6_sorted.part | xargs -I {} ipset $ADD privacy-update_ipv6 {}
   nice -n 2 ipset $SWAP privacy-update_ipv6 privacy-filter_ipv6
   nice -n 2 ipset $DESTROY privacy-update_ipv6
fi
iptables -L | grep privacy-filter_ipv6 > /dev/null 2>&1
if [ $? -ne 0 ]; then
   nice -n 2 ip6tables -I FORWARD -m set $MATCH_SET privacy-filter_ipv6 src,dst -j $fwoption
else
   nice -n 2 ip6tables -D FORWARD -m set $MATCH_SET privacy-filter_ipv6 src,dst -j $fwoption
   nice -n 2 ip6tables -I FORWARD -m set $MATCH_SET privacy-filter_ipv6 src,dst -j $fwoption
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

logipv6 () {
logger -s -t system "Privacy Filter (ipv6) loaded $(ipset -L  privacy-filter_ipv6 | wc -l | awk '{print $1-7}') unique ip addresses."
}

cleanup () {
find /tmp -name 'privacy-filter_ipv*.part' -exec rm {} +
logger -s -t system "Privacy Filter (ipv4) loaded $(ipset -L  privacy-filter_ipv4 | wc -l | awk '{print $1-7}') unique ip addresses."
check_ipv6 logipv6
}

check_online
fix_list
run_blocklists
run_ipset
cleanup

exit $?
```
Note: save this list as [privacy-filter.list](https://gitlab.com/swe_toast/privacy-filter/raw/master/privacy-filter.list) in your path on the router, if you set this file in the wrong place the script will automatically download a new copy and set it at either path or failover path.

