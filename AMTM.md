## About ##
**amtm - the Asuswrt-Merlin Terminal Menu**

amtm is a shell-based front end that manages popular scripts for wireless routers running Asuswrt-Merlin firmware.

Starting with Asuswrt-Merlin 384.15, amtm is included in the firmware.

amtm is maintained by [@decoderman](https://github.com/decoderman), aka thelonelycoder. [GitHub repo](https://github.com/decoderman/amtm), [Website](https://diversion.ch/amtm.html).  
There are [support threads](https://www.snbforums.com/threads/amtm-4-2-the-asuswrt-merlin-terminal-menu-released-january-13-2024.79665/) at SNBForums that are dedicated to amtm.

 
## Features ##

amtm currently supports these popular scripts:

| Script | Maintainer | Infos |
|--------|------------|------|
| Diversion | thelonelycoder | [Link](https://www.snbforums.com/forums/asuswrt-merlin-addons.60/?prefix_id=10&starter_id=25480) |
| Skynet | Adamm | [Link](https://www.snbforums.com/forums/asuswrt-merlin-addons.60/?prefix_id=14) |
| FlexQoS | dave14305 | [Link](https://www.snbforums.com/forums/asuswrt-merlin-addons.60/?prefix_id=8&starter_id=58901) |
| YazFi | Jack Yaz | [Link](https://www.snbforums.com/forums/asuswrt-merlin-addons.60/?prefix_id=13&starter_id=53009) |
| scribe | cmkelley | [Link](https://www.snbforums.com/forums/asuswrt-merlin-addons.60/?prefix_id=7) |
| x3mRouting | Xentrk | [Link](https://www.snbforums.com/forums/asuswrt-merlin-addons.60/?prefix_id=9) |
| unbound Manager | Martineau | [Link](https://www.snbforums.com/forums/asuswrt-merlin-addons.60/?prefix_id=5) |
| connmon |Jack Yaz | [Link](https://www.snbforums.com/forums/asuswrt-merlin-addons.60/?prefix_id=18&starter_id=53009) |
| ntpMerlin | Jack Yaz | [Link](https://www.snbforums.com/forums/asuswrt-merlin-addons.60/?prefix_id=22&starter_id=53009) |
| scMerlin | Jack Yaz | [Link](https://www.snbforums.com/forums/asuswrt-merlin-addons.60/?prefix_id=23&starter_id=53009) |
| spdMerlin | Jack Yaz | [Link](https://www.snbforums.com/forums/asuswrt-merlin-addons.60/?prefix_id=19&starter_id=53009) |
| uiDivStats | Jack Yaz | [Link](https://www.snbforums.com/forums/asuswrt-merlin-addons.60/?prefix_id=15&starter_id=53009) |
| uiScribe | Jack Yaz | [Link](https://www.snbforums.com/forums/asuswrt-merlin-addons.60/?prefix_id=24&starter_id=53009) |
| DNSCrypt | bigeyes0x0, SomeWhereOverTheRainBow | [Link](https://www.snbforums.com/forums/asuswrt-merlin-addons.60/?prefix_id=29&starter_id=64179) |
| Pixelserv-tls | kvic | [Link](https://www.snbforums.com/forums/asuswrt-merlin-addons.60/?prefix_id=17) |
| YazDHCP | Jack Yaz | [Link](https://www.snbforums.com/forums/asuswrt-merlin-addons.60/?prefix_id=31&starter_id=53009) |
| Vnstat | dev_null | [Link](https://www.snbforums.com/threads/beta-2-vnstat-on-merlin-ui-cli-and-email-data-use-monitoring-with-full-install-and-menu.70727/) |
| WireGuard Session Manager | Martineau | [Link](https://www.snbforums.com/forums/asuswrt-merlin-addons.60/?prefix_id=32&starter_id=13215) |
| Asuswrt-Merlin-AdGuardHome-Installer | SomeWhereOverTheRainBow | [Link](https://www.snbforums.com/threads/new-release-asuswrt-merlin-adguardhome-installer.76506/) |
| VPNMON-R2 | Viktor Jaep | [Link](https://www.snbforums.com/forums/asuswrt-merlin-addons.60/?prefix_id=36) |
| RTRMON | Viktor Jaep | [Link](https://www.snbforums.com/forums/asuswrt-merlin-addons.60/) |
| WICENS | Maverickcdn | [Link](https://www.snbforums.com/threads/wicens-wan-ip-change-email-notification-script.69294/) |
| KILLMON | Viktor Jaep | [Link](https://www.snbforums.com/threads/killmon-v1-05-feb-20-2023-ip4-ip6-vpn-kill-switch-monitor-configurator.81758/) |
| Dual WAN Failover | Ranger802004 | [Link](https://www.snbforums.com/threads/dual-wan-failover-v2-0-2-release.83674/) |
| BACKUPMON | Viktor Jaep | [Link](https://www.snbforums.com/threads/backupmon-v1-22-oct-2-2023-backup-restore-your-router-jffs-nvram-external-usb-drive.86645/) |
| Domain-based VPN Routing | Ranger802004 | [Link](https://www.snbforums.com/threads/domain-based-vpn-routing-script.79264/) |

amtm also offers an interface for managing a number of other features:

| Other features | Maintainer |
|----------------|-----------|
| Entware | zyxmon, ryzhovau, themiron |
| USB disk check at boot | ColinTaylor, latenitetech, thelonelycoder |
| Format disk | thelonelycoder, ColinTaylor |
| Router LED control, smart router LED scheduler | thelonelycoder |
| Reboot scheduler via cron job | thelonelycoder |
| Swap file creation and management | thelonelycoder |
| amtm themes | thelonelycoder |
| email settings | thelonelycoder |
| 10 router games to choose from | thelonelycoder |
| Keep a history of entered shell commands | thelonelycoder
| Router date keeper | thelonelycoder
| amtm and third party script reset/remove options | thelonelycoder
| Show all cron jobs | thelonelycoder
| Firmware update notification | thelonelycoder
| Reboot router command | thelonelycoder
| Scripts update notification | thelonelycoder

## Usage ##
amtm can be launched over SSH, as there is no web interface for it.  In addition to enabling SSH support on your router, some of its scripts will have additional requirements, most commonly you will need to set _Enable JFFS custom scripts and configs_ on the web interface to _Yes_, under _Administration -> System_.  A USB disk is also required by many of its scripts.

Connect to your router using an SSH client (Xshell, putty, etc...), then launch amtm by simply typing 
```
amtm
```
in the console.  A menu will appear, guiding you through the various options available.  Installing Entware is usually the first step you should do, since many scripts will require it.

Before installing any of the scripts it is strongly recommended that you read the documentation for these scripts.


## amtm License ##
amtm is free to use under the GNU General Public License, version 3 (GPL-3.0)

## Screenshot amtm 4.2 ##
[![amtm 4.2](https://cdn.imgchest.com/files/e4gdc5dxw24.png "amtm 4.2")](https://cdn.imgchest.com/files/e4gdc5dxw24.png "amtm 4.2")