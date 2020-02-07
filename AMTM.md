## About ##
amtm is a front end that manages popular scripts for wireless routers running Asuswrt-Merlin firmware.

Starting with Asuswrt-Merlin 384.15, amtm is included in the firmware.

amtm is intended to be a helper script, a convenient shortcut manager to install and manage various third party scripts on your router.

There is a [support thread](https://www.snbforums.com/threads/amtm-the-asuswrt-merlin-terminal-menu.42415/) at SNBForums that is dedicated to amtm.

 
## Features ##

amtm currently supports these popular scripts:

| Script | Maintainer | Infos |
|--------|------------|------|
| Diversion | thelonelycoder | [Link](https://www.snbforums.com/threads/diversion-the-router-ad-blocker.48538/) |
| Skynet | Adamm | [Link](https://www.snbforums.com/threads/skynet-asus-firewall-addition-dynamic-malware-country-manual-ip-blocking.16798/) |
| FreshJR Adaptive QOS | FreshJR | [Link](https://www.snbforums.com/threads/release-freshjr-adaptive-qos-improvements-custom-rules-and-inner-workings.36836/) |
| YazFi | Jack Yaz | [Link](https://www.snbforums.com/threads/yazfi-enhanced-asuswrt-merlin-guest-wifi-inc-ssid-vpn-client.45924/) |
| scribe | cmkelley | [Link](https://www.snbforums.com/threads/scribe-syslog-ng-and-logrotate-installer.55853/) |
| x3mRouting | Xentrk | [Link](https://www.snbforums.com/threads/x3mrouting-selective-routing-for-asuswrt-merlin-firmware.57793/) |
| connmon |Jack Yaz | [Link](https://www.snbforums.com/threads/connmon-internet-connection-monitoring.56163/) |
| ntpMerlin | Jack Yaz | [Link](https://www.snbforums.com/threads/ntpmerlin-ntp-daemon-for-asuswrt-merlin.55756/) |
| scMerlin | Jack Yaz | [Link](https://www.snbforums.com/threads/scmerlin-service-and-script-control-menu-for-asuswrt-merlin.56277/) |
| spdMerlin | Jack Yaz | [Link](https://www.snbforums.com/threads/spdmerlin-automated-speedtests-with-graphs.55904/) |
| uiDivStats | Jack Yaz | [Link](https://www.snbforums.com/threads/uidivstats-webui-for-diversion-statistics.56393/) |
| uiScribe | Jack Yaz | [Link](https://www.snbforums.com/threads/uiscribe-custom-system-log-page-for-scribed-logs.57040/) |
| Stubby DNS | Xentrk and Adamm | _deprecated_ |
| DNSCrypt | bigeyes0x0 | [Link](https://www.snbforums.com/threads/release-dnscrypt-installer-for-asuswrt.36071/) |
| Pixelserv-tls | kvic | [Link](https://www.snbforums.com/threads/pixelserv-a-better-one-pixel-webserver-for-adblock.26114/) |


amtm also offers an interface for managing a number of other features:

| Other features | Maintainer |
|----------------|-----------|
| Entware _(replaces the old entware-setup.sh)_ | zyxmon, ryzhovau, themiron |
| USB disk check at boot | ColinTaylor, latenitetech, thelonelycoder |
| Format disk | thelonelycoder, ColinTaylor |
| Reboot scheduler via cron job | thelonelycoder |
| Swap file creation and management | thelonelycoder |
| amtm themes | thelonelycoder |

## Usage ##
amtm can be launched over SSH, as there is no web interface for it.  In addition to enabling SSH support on your router, some of its scripts will have additional requirements, most commonly you will need to set _Enable JFFS custom scripts and configs_ on the web interface to _Yes_, under _Administration -> System_.  A USB disk is also required by many of its scripts.

Connect to your router using an SSH client (Xshell, putty, etc...), then launch amtm by simply typing "amtm" in the console.  A menu will appear, guiding you through the various options available.  Installing Entware is usually the first step you should do, since many scripts will require it.

Before installing any of the scripts it is strongly recommended that you read the documentation for these scripts.


## amtm License ##
amtm is free to use under the GNU General Public License, version 3 (GPL-3.0)