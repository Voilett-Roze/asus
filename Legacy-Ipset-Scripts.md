> These scripts are considered legacy scripts, they have no maintainers and getting support on them might get tricky, they also only supports ipset version 4 so if you have a new router these scripts will not work, please consult the chart on [here](https://github.com/RMerl/asuswrt-merlin/wiki/Using-ipset)

# Peer Guardian

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
***
# Disable Windows10 Tracking
_Please note, this how-to will work only on asuswrt-merlin releases newer than 378.55. 380.57 or newer is recommended._

This solution consists of two parts:

* collecting resolved IPs from unwanted domains list to IP set with dnsmasq,
* dropping traffic from/to collected IPs with firewall rule.

First, enable JFFS custom scripts and configs from WebUI. Put following list of unwanted domains to `/jffs/configs/windows-10-tracking-hosts.txt`:
```
a.ads1.msn.com
a.ads2.msads.net
a.ads2.msn.com
a.rad.msn.com
a-0001.a-msedge.net
a-0002.a-msedge.net
a-0003.a-msedge.net
a-0004.a-msedge.net
a-0005.a-msedge.net
a-0006.a-msedge.net
a-0007.a-msedge.net
a-0008.a-msedge.net
a-0009.a-msedge.net
ac3.msn.com
ad.doubleclick.net
adnexus.net
adnxs.com
ads.msn.com
ads1.msads.net
ads1.msn.com
aidps.atdmt.com
aka-cdn-ns.adtech.de
a-msedge.net
apps.skype.com
az361816.vo.msecnd.net
az512334.vo.msecnd.net
b.ads1.msn.com
b.ads2.msads.net
b.rad.msn.com
bs.serving-sys.com
c.atdmt.com
c.msn.com
cdn.atdmt.com
cds26.ams9.msecn.net
choice.microsoft.com
choice.microsoft.com.nsatc.net
compatexchange.cloudapp.net
corp.sts.microsoft.com
corpext.msitadfs.glbdns2.microsoft.com
cs1.wpc.v0cdn.net
db3aqu.atdmt.com
df.telemetry.microsoft.com
diagnostics.support.microsoft.com
ec.atdmt.com
fe2.update.microsoft.com.akadns.net
feedback.microsoft-hohm.com
feedback.search.microsoft.com
feedback.windows.com
flex.msn.com
g.msn.com
h1.msn.com
i1.services.social.microsoft.com
i1.services.social.microsoft.com.nsatc.net
lb1.www.ms.akadns.net
live.rads.msn.com
m.adnxs.com
m.hotmail.com
msedge.net
msftncsi.com
msnbot-65-55-108-23.search.msn.com
msntest.serving-sys.com
oca.telemetry.microsoft.com
oca.telemetry.microsoft.com.nsatc.net
pre.footprintpredict.com
preview.msn.com
pricelist.skype.com
rad.live.com
rad.msn.com
redir.metaservices.microsoft.com
reports.wes.df.telemetry.microsoft.com
s.gateway.messenger.live.com
s0.2mdn.net
schemas.microsoft.akadns.net
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
```
I took mine list from [here](http://forums.zyxmon.org/go.php?https://github.com/10se1ucgo/DisableWinTracking/blob/master/run.py#L208). Now put following content to `/jffs/scripts/firewall-start`:
```
#!/bin/sh
JFFS_CONFIG_DIR=/jffs/configs
BLOCKED_HOSTS_FILE=${JFFS_CONFIG_DIR}/windows-10-tracking-hosts.txt
DNSMASQ_CFG=${JFFS_CONFIG_DIR}/dnsmasq.conf.add
if [ ! -f $DNSMASQ_CFG ] || [ "$(grep Win10tracking $DNSMASQ_CFG)" = "" ];
then
  rm -f $DNSMASQ_CFG
  for i in `cat $BLOCKED_HOSTS_FILE`;
  do
    echo "ipset=/$i/Win10tracking" >> $DNSMASQ_CFG
  done
  service restart_dnsmasq
fi

# Load ipset modules
lsmod | grep "ipt_set" > /dev/null 2>&1 || \
for module in ip_set ip_set_nethash ip_set_iphash ipt_set
do
    insmod $module
done

# Create ip set
if [ "$(ipset --swap Win10tracking Win10tracking 2>&1 | grep 'Unknown set')" != "" ];
then
  ipset -N Win10tracking iphash
fi

# Apply iptables rule
iptables-save | grep Win10tracking > /dev/null 2>&1 && exit
case $(uname -m) in
  armv7l)
    iptables -I FORWARD -m set --match-set Win10tracking src,dst -j DROP
    ;;
  mips)
    iptables -I FORWARD -m set --set Win10tracking src,dst -j DROP
    ;;
esac
```
Don't forget to make script executable and reboot router to take effect:
```
chmod +x /jffs/scripts/firewall-start
reboot
```
You may check it's working by trying to open some site from list (view.atdmt.com for example). Then check "black list" is populated with some IP addresses:
```
ipset --list Win10tracking
```

Alternatively, put the above script into `/jffs/scripts/windows-10-tracking-blocker` and call that from `/jffs/scripts/firewall-start`.