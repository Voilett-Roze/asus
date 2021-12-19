## About ##
**amtm - the Asuswrt-Merlin Terminal Menu**

amtm is a shell-based front end that manages popular scripts for wireless routers running Asuswrt-Merlin firmware.  It also offers a couple of additional management features.

Starting with Asuswrt-Merlin 384.15, amtm is included in the firmware.

There is a [support thread](https://www.snbforums.com/threads/amtm-the-asuswrt-merlin-terminal-menu.42415/) at SNBForums that is dedicated to amtm.

 
## Features ##

amtm currently supports these popular scripts:

| Script | Maintainer | Infos |
|--------|------------|------|
| Diversion | thelonelycoder | [Link](https://www.snbforums.com/forums/asuswrt-merlin-addons.60/?prefix_id=10&starter_id=25480) |
| Skynet | Adamm | [Link](https://www.snbforums.com/forums/asuswrt-merlin-addons.60/?prefix_id=14) |
| FlexQoS | dave14305 | [Link](https://www.snbforums.com/forums/asuswrt-merlin-addons.60/?prefix_id=8&starter_id=58901) |
| FreshJR Adaptive QOS | FreshJR | [Link](https://www.snbforums.com/threads/release-freshjr-adaptive-qos-improvements-custom-rules-and-inner-workings.36836/) deprecated |
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
| Stubby DNS | Xentrk and Adamm | _deprecated_ |
| DNSCrypt | bigeyes0x0, SomeWhereOverTheRainBow | [Link](https://www.snbforums.com/forums/asuswrt-merlin-addons.60/?prefix_id=29&starter_id=64179) |
| Pixelserv-tls | kvic | [Link](https://www.snbforums.com/forums/asuswrt-merlin-addons.60/?prefix_id=17) |
| NVRAM Save/Restore Utility | Xentrk, John9527 & others | [Link](https://www.snbforums.com/forums/asuswrt-merlin-addons.60/?prefix_id=28) |
| YazDHCP | Jack Yaz | [Link](https://www.snbforums.com/forums/asuswrt-merlin-addons.60/?prefix_id=31&starter_id=53009) |
| Vnstat | dev_null | [Link](https://www.snbforums.com/threads/beta-2-vnstat-on-merlin-ui-cli-and-email-data-use-monitoring-with-full-install-and-menu.70727/) |
| WireGuard Session Manager | Martineau | [Link](https://www.snbforums.com/forums/asuswrt-merlin-addons.60/?prefix_id=32&starter_id=13215) |

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

## Usage ##
amtm can be launched over SSH, as there is no web interface for it.  In addition to enabling SSH support on your router, some of its scripts will have additional requirements, most commonly you will need to set _Enable JFFS custom scripts and configs_ on the web interface to _Yes_, under _Administration -> System_.  A USB disk is also required by many of its scripts.

Connect to your router using an SSH client (Xshell, putty, etc...), then launch amtm by simply typing "amtm" in the console.  A menu will appear, guiding you through the various options available.  Installing Entware is usually the first step you should do, since many scripts will require it.

Before installing any of the scripts it is strongly recommended that you read the documentation for these scripts.


## amtm License ##
amtm is free to use under the GNU General Public License, version 3 (GPL-3.0)

## Screenshot amtm v3.1.1, FW ##
[![amtm v3.1.1](https://i.imgur.com/3qYg2cL.png "amtm v3.1.1")](https://i.imgur.com/3qYg2cL.png "amtm v3.1.1")