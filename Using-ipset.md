# Using ipset

Since 3.0.0.4_270.26 [ipset](http://en.wikipedia.org/wiki/Netfilter#ipset) feature has been implemented. This is an Netfilter extension which able:
* to store multiple IP addresses or port numbers and match against the collection by iptables at one swoop;
* to dynamically update iptables rules against IP addresses or ports without performance penalty;
* to express complex IP address and ports based rulesets with one single iptables rule and benefit from the speed of IP sets.

> **NOTE:** _Most scripts on this page supports only IPSET 4.x that will result in scripts not working on newer routers with IPSET 6.x_

# Tor and Countries Block
Supports only IPSET 4 

This is an example of using [ipset utility](http://manpages.ubuntu.com/manpages/lucid/man8/ipset.8.html) with two different set types: iphash and nethash. The example shows how to block incoming connection form [Tor](https://www.torproject.org/) nodes (iphash set type — number of ip addresses) and how to block incoming connection from whole countries (nethash set type - number of ip subnets). 

Please, enable and format [JFFS](https://github.com/RMerl/asuswrt-merlin/wiki/JFFS) through WEB UI, place this content to `/jffs/scripts/firewall-start`

```
#!/bin/sh

# Loading ipset modules
lsmod | grep "ipt_set" > /dev/null 2>&1 || \
for module in ip_set ip_set_nethash ip_set_iphash ipt_set
do
    insmod $module
done

# Preparing folder to cache downloaded files
IPSET_LISTS_DIR=/jffs/ipset_lists
[ -d "$IPSET_LISTS_DIR" ] || mkdir -p $IPSET_LISTS_DIR

# Different routers got different iptables syntax
case $(uname -m) in
  armv7l)
    MATCH_SET='--match-set'
    ;;
  mips)
    MATCH_SET='--set'
    ;;
esac

# Block traffic from Tor nodes
if [ "$(ipset --swap TorNodes TorNodes 2>&1 | grep 'Unknown set')" != "" ]
then
    ipset -N TorNodes iphash
    [ -e $IPSET_LISTS_DIR/tor.lst ] || wget -q -O $IPSET_LISTS_DIR/tor.lst http://torstatus.blutmagie.de/ip_list_all.php/Tor_ip_list_ALL.csv
    for IP in $(cat $IPSET_LISTS_DIR/tor.lst)
    do
        ipset -A TorNodes $IP
    done
fi
[ -z "$(iptables-save | grep TorNodes)" ] && iptables -I INPUT -m set $MATCH_SET TorNodes src -j DROP

# Block incoming traffic from some countries. cn and pk is for China and Pakistan. See other countries code at http://www.ipdeny.com/ipblocks/
if [ "$(ipset --swap BlockedCountries BlockedCountries 2>&1 | grep 'Unknown set')" != "" ]
then
    ipset -N BlockedCountries nethash
    for country in pk cn
    do
        [ -e $IPSET_LISTS_DIR/$country.lst ] || wget -q -O $IPSET_LISTS_DIR/$country.lst http://www.ipdeny.com/ipblocks/data/countries/$country.zone
        for IP in $(cat $IPSET_LISTS_DIR/$country.lst)
        do
            ipset -A BlockedCountries $IP
        done
    done
fi
[ -z "$(iptables-save | grep BlockedCountries)" ] && iptables -I INPUT -m set $MATCH_SET BlockedCountries src -j DROP

# Block Microsoft telemetry spying servers
if [ "$(ipset --swap MicrosoftSpyServers MicrosoftSpyServers 2>&1 | grep 'Unknown set')" != "" ]
then
    ipset -N MicrosoftSpyServers iphash
    for IP in 23.99.10.11 63.85.36.35 63.85.36.50 64.4.6.100 64.4.54.22 64.4.54.32 64.4.54.254 \
              65.52.100.7 65.52.100.9 65.52.100.11 65.52.100.91 65.52.100.92 65.52.100.93 65.52.100.94 \
              65.55.29.238 65.55.39.10 65.55.44.108 65.55.163.222 65.55.252.43 65.55.252.63 65.55.252.71 \
              65.55.252.92 65.55.252.93 66.119.144.157 93.184.215.200 104.76.146.123 111.221.29.177 \
              131.107.113.238 131.253.40.37 134.170.52.151 134.170.58.190 134.170.115.60 134.170.115.62 \
              134.170.188.248 157.55.129.21 157.55.133.204 157.56.91.77 168.62.187.13 191.234.72.183 \
              191.234.72.186 191.234.72.188 191.234.72.190 204.79.197.200 207.46.223.94 207.68.166.254
    do
        ipset -A MicrosoftSpyServers $IP
    done
fi
[ -z "$(iptables-save | grep MicrosoftSpyServers)" ] && iptables -I FORWARD -m set $MATCH_SET MicrosoftSpyServers dst -j DROP
```
and make it executable:
```
chmod +x /jffs/scripts/firewall-start
```
You may run `/jffs/scripts/firewall-start` from command line or reboot router to apply new blocking rules immediately. 


***
## Peer Guardian
Supports only IPSET 4 

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

its recommended not to store this script in firewall-start rather add the script to /opt/bin/malware-block 

then type this

> nano /jffs/scripts/services-start 

and append

> cru a malware-filter "0 */12 * * * /opt/bin/malware-block"

save it this will make malware-block run every 12th hour and update the router.
```
#!/bin/sh
# Author: Toast
# Contributers: Octopus, Tomsk, Neurophile, jimf, spalife
# Testers: shooter40sw
# Revision 10

path=/opt/var/cache/malware-filter                      # Set your path here
retries=3                                               # Set number of tries here
regexp=`echo "\b([0-9]{1,3}\.){3}[0-9]{1,3}\b"`         # Dont change this value

case $(ipset -v | grep -oE "ipset v[0-9]") in
*v6) # Value for ARM Routers

    MATCH_SET='--match-set'
    HASH='hash:ip'
    SYNTAX='add'
    SWAPPED='swap'
    DESTROYED='destroy'
    OPTIONAL='family inet hashsize 2048 maxelem 65536'

     ipsetv=6
     lsmod | grep "xt_set" > /dev/null 2>&1 || \
     for module in ip_set ip_set_hash_net ip_set_hash_ip xt_set
     do
          insmod $module
     done
;;

*v4) # Value for Mips Routers

    MATCH_SET='--set'
    HASH='iphash'
    SYNTAX='-q -A'
    SWAPPED='-W'
    DESTROYED='--destroy'
    OPTIONAL=''

    ipsetv=4
     lsmod | grep "ipt_set" > /dev/null 2>&1 || \
     for module in ip_set ip_set_nethash ip_set_iphash ipt_set
     do
          insmod $module
     done
;;
esac

get_list () {
        mkdir -p $path
        wget -q --tries=$retries --show-progress -i $path/malware-filter.list -O $path/malware-list.pre
        cat $path/malware-list.pre | grep -oE "$regexp" | sort -u >$path/malware-filter.txt
 }

run_ipset () {

get_list

echo "adding ipset rule to firewall this will take time."

ipset -L malware-filter >/dev/null 2>&1
if [ $? -ne 0 ]; then
    if [ "$(ipset --swap malware-filter malware-filter 2>&1 | grep -E 'Unknown set|The set with the given name does not exist')" != "" ]; then
    nice -n 2 ipset -N malware-filter $HASH $OPTIONAL
    if [ -f /opt/bin/xargs ]; then
    /opt/bin/xargs -P10 -I "PARAM" -n1 -a $path/malware-filter.txt nice -n 2 ipset $SYNTAX malware-filter PARAM
    else for i in `cat $path/malware-filter.txt`; do nice -n 2 ipset $SYNTAX malware-filter $i ; done; fi
fi
else
    nice -n 2 ipset -N malware-update $HASH $OPTIONAL
    if [ -f /opt/bin/xargs ]; then
    /opt/bin/xargs -P10 -I "PARAM" -n1 -a $path/malware-filter.txt nice -n 2 ipset $SYNTAX malware-update PARAM
    else for i in `cat $path/malware-filter.txt`; do nice -n 2 ipset $SYNTAX malware-update $i ; done; fi
    nice -n 2 ipset $SWAPPED malware-update malware-filter
    nice -n 2 ipset $DESTROYED malware-update
fi

iptables -L | grep malware-filter > /dev/null 2>&1
if [ $? -ne 0 ]; then
    nice -n 2 iptables -I FORWARD -m set $MATCH_SET malware-filter src,dst -j REJECT
else
    nice -n 2 iptables -D FORWARD -m set $MATCH_SET malware-filter src,dst -j REJECT
    nice -n 2 iptables -I FORWARD -m set $MATCH_SET malware-filter src,dst -j REJECT
fi
}

run_ipset

logger -s -t system "Malware Filter loaded $(cat $path/malware-filter.txt | wc -l) unique ip addresses."
exit $?
```
Save this list as malware-filter.list and set it in your relative path (see configuration part in script) you can also add more list by just appending to this list.
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

So this script tries to block [Telemetry](http://www.zdnet.com/article/windows-10-telemetry-secrets/) and some additional Google Servers and some Chinese data collection centers for [Android rootkits](http://arstechnica.com/security/2016/11/powerful-backdoorrootkit-found-preinstalled-on-3-million-android-phones/)

```
#!/bin/sh
# Author: Toast
# Contributers: Tomsk
# Revision 4

path=/opt/var/cache/privacy-filter                      # Set your path here
dnsmasq_cfg=/jffs/configs/dnsmasq.conf.add
dnsmasq_real=/tmp/etc/dnsmasq.conf
regexp=`echo "\b([0-9]{1,3}\.){3}[0-9]{1,3}\b"`         # Dont change this value

if  grep -Fxq "#privacy-filter" $dnsmasq_real
    then    logger -s -t privacy-filter is present in $dnsmasq_real
    else    echo "#privacy-filter" >> $dnsmasq_cfg
            for i in `cat $path/privacy-filter.list`
            do echo "server=/$i/127.0.0.1#1919" >> $dnsmasq_cfg; done
            service restart_dnsmasq
            logger -s -t privacy-filter was added to $dnsmasq_cfg; fi

case $(ipset -v | grep -oE "ipset v[0-9]") in
*v6) # Value for ARM Routers

    MATCH_SET='--match-set'
    HASH='hash:ip'
    SYNTAX='add'
    SWAPPED='swap'
    DESTROYED='destroy'
    OPTIONAL='family inet hashsize 2048 maxelem 65536'

     ipsetv=6
     lsmod | grep "xt_set" > /dev/null 2>&1 || \
     for module in ip_set ip_set_hash_net ip_set_hash_ip xt_set
     do
          insmod $module
     done
;;

*v4) # Value for Mips Routers

    MATCH_SET='--set'
    HASH='iphash'
    SYNTAX='-q -A'
    SWAPPED='-W'
    DESTROYED='--destroy'
    OPTIONAL=''

     ipsetv=4
     lsmod | grep "ipt_set" > /dev/null 2>&1 || \
     for module in ip_set ip_set_nethash ip_set_iphash ipt_set
     do
          insmod $module
     done
;;
esac


run_ipset () {

ipset -L privacy-filter >/dev/null 2>&1
if [ $? -ne 0 ]; then
    if [ "$(ipset --swap privacy-filter privacy-filter 2>&1 | grep -E 'Unknown set|The set with the given name does not exist')" != "" ]; then
    nice ipset -N privacy-filter $HASH $OPTIONAL
    for i in `cat $path/privacy-filter.txt`; do nice -n 2 ipset $SYNTAX privacy-filter $i ; done
fi
else
    nice -n 2 ipset -N privacy-update $HASH $OPTIONAL
    for i in `cat $path/privacy-filter.txt`; do nice -n 2 ipset $SYNTAX privacy-update $i ; done
    nice -n 2 ipset $SWAPPED privacy-update privacy-filter
    nice -n 2 ipset $DESTROYED privacy-update
fi

iptables -L | grep privacy-filter > /dev/null 2>&1
if [ $? -ne 0 ]; then
    nice -n 2 iptables -I FORWARD -m set $MATCH_SET privacy-filter src,dst -j REJECT
else
    nice -n 2 iptables -D FORWARD -m set $MATCH_SET privacy-filter src,dst -j REJECT
    nice -n 2 iptables -I FORWARD -m set $MATCH_SET privacy-filter src,dst -j REJECT
fi
}

run_ipset
exit $?
```

and here is the iplist of the telemertry server taken of the original script save as privacy-filter.txt in your path

```
23.99.10.11
63.85.36.35
63.85.36.50
64.4.6.100
64.4.54.22
64.4.54.32
64.4.54.254
65.52.100.7
65.52.100.9
65.52.100.11
65.52.100.91
65.52.100.92
65.52.100.93
65.52.100.94
65.55.29.238
65.55.39.10
65.55.44.108
65.55.163.222
65.55.252.43
65.55.252.63
65.55.252.71
65.55.252.92
65.55.252.93
66.119.144.157
93.184.215.200
104.76.146.123
111.221.29.177
131.107.113.238
131.253.40.37
134.170.52.151
134.170.58.190
134.170.115.60
134.170.115.62
134.170.188.248
157.55.129.21
157.55.133.204
157.56.91.77
168.62.187.13
191.234.72.183
191.234.72.186
191.234.72.188
191.234.72.190
204.79.197.200
207.46.223.94
207.68.166.254
```

and last but not least the list save as privacy-filter.list in your path

```
googleadservices.com
www.google-analytics.com
google-analytics.com
ssl.google-analytics.com
secure.adnxs.com
secure.flashtalking.com
services.wes.df.telemetry.microsoft.com
settings-sandbox.data.microsoft.com
settings-win.data.microsoft.com
sls.update.microsoft.com.akadns.net
sqm.df.telemetry.microsoft.com
sqm.telemetry.microsoft.com
sqm.telemetry.microsoft.com.nsatc.net
static.2mdn.net
statsfe1.ws.microsoft.com
statsfe2.update.microsoft.com.akadns.net
statsfe2.ws.microsoft.com
survey.watson.microsoft.com
telecommand.telemetry.microsoft.com
telecommand.telemetry.microsoft.com.nsatc.net
telemetry.appex.bing.net
telemetry.microsoft.com
telemetry.urs.microsoft.com
view.atdmt.com
vortex.data.microsoft.com
vortex-bn2.metron.live.com.nsatc.net
vortex-cy2.metron.live.com.nsatc.net
vortex-sandbox.data.microsoft.com
vortex-win.data.microsoft.com
watson.live.com
watson.microsoft.com
watson.ppe.telemetry.microsoft.com
watson.telemetry.microsoft.com
watson.telemetry.microsoft.com.nsatc.net
wes.df.telemetry.microsoft.com
www.msftncsi.com
nametests.com
oyag.lhzbdvm.com
oyag.prugskh.net
oyag.prugskh.com
```
## Shoblock
* Supports both IPSET 4 and 6
* Requirement Entware package: hostip

Blocks known ip addresses from shodan.io scanners this script populates with dns if they are not added initially and more can be added by adding to /opt/var/cache/shoblock/shodandns.list

```
#!/bin/sh
# Author: Toast
# Revision 1
path=/opt/var/cache/shoblock
url=https://gitlab.com/swe_toast/shodan-block/raw/master/shodandns.list
get_list () {
if [ -z "$(which opkg)" ]; then logger -s -t ublockr "no package manager found"; exit 0; else
	if [ -z "$(opkg list-installed | grep hostip)" ]; then opkg install hostip; fi
	if [ -f $path/block.list ]; then rm $path/block.list; fi
	if [ -z $path/shodandns.list]; then
        mkdir -p $path
        wget -q --tries=$retries --show-progress $url -O $path/shodandns.list
    fi
	for i in `cat $path/shodandns.list`; do hostip $i >>$path/block.pre; done
	sort -u $path/block.pre > $path/block.list
	if [ -f $path/block.pre ]; then rm $path/block.pre; fi
fi }
case $(ipset -v | grep -oE "ipset v[0-9]") in
*v6) # Value for ARM Routers
    MATCH_SET='--match-set'
    HASH='hash:ip'
    SYNTAX='add'
    SWAPPED='swap'
    DESTROYED='destroy'
     ipsetv=6
     lsmod | grep "xt_set" > /dev/null 2>&1 || \
     for module in ip_set ip_set_hash_net ip_set_hash_ip xt_set
     do
          insmod $module
     done
;;
*v4) # Value for Mips Routers
    MATCH_SET='--set'
    HASH='iphash'
    SYNTAX='-q -A'
    SWAPPED='-W'
    DESTROYED='--destroy'
    OPTIONAL=''
    ipsetv=4
     lsmod | grep "ipt_set" > /dev/null 2>&1 || \
     for module in ip_set ip_set_nethash ip_set_iphash ipt_set
     do
          insmod $module
     done
;;
esac
run_ipset () {
get_list
echo "adding ipset rule to firewall this will take time."
ipset -L shodan-block >/dev/null 2>&1
if [ $? -ne 0 ]; then
    if [ "$(ipset --swap shodan-block shodan-block 2>&1 | grep -E 'Unknown set|The set with the given name does not exist')" != "" ]; then
    nice -n 2 ipset -N shodan-block $HASH
        if [ -f /opt/bin/xargs ]; then
        /opt/bin/xargs -P10 -I "PARAM" -n1 -a $path/block.list nice -n 2 ipset $SYNTAX shodan-block PARAM
        else for i in `cat $path/block.list`; do nice -n 2 ipset $SYNTAX shodan-block $i ; done; fi
fi
else
    nice -n 2 ipset -N shodan-update $HASH
	if [ -f /opt/bin/xargs ]; then
        /opt/bin/xargs -P10 -I "PARAM" -n1 -a $path/block.list nice -n 2 ipset $SYNTAX shodan-update PARAM
        else for i in `cat $path/block.list`; do nice -n 2 ipset $SYNTAX shodan-update $i ; done; fi
        nice -n 2 ipset $SWAPPED shodan-update shodan-block
    nice -n 2 ipset $DESTROYED shodan-update
fi
iptables -L | grep shodan-block > /dev/null 2>&1
if [ $? -ne 0 ]; then
    nice -n 2 iptables -I FORWARD -m set $MATCH_SET shodan-block src,dst -j REJECT
else
    nice -n 2 iptables -D FORWARD -m set $MATCH_SET shodan-block src,dst -j REJECT
    nice -n 2 iptables -I FORWARD -m set $MATCH_SET shodan-block src,dst -j REJECT
fi
}
run_ipset
logger -s -t system "Shodan Scanner Filter loaded $(cat $path/shodandns.list | wc -l) unique ip addresses to the blocklist."
exit $?
```
