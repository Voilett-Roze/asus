This is the official Wiki/documentation for Asuswrt-merlin, a custom firmware designed for Asus routers.

>Note: _As with any Wiki, this documentation is a constant work-in-progress.  Most of the content is contributed by the community - anyone with a Github account can edit it._

### About
1. [About Asuswrt and Asuswrt-merlin](/RMerl/asuswrt-merlin.ng/wiki/About-Asuswrt/)
2. [Features](https://www.asuswrt-merlin.net/features) (External link)
3. [Screenshots](https://www.asuswrt-merlin.net/screenshots) (External link)
4. [Supported devices](/RMerl/asuswrt-merlin.ng/wiki/Supported-Devices)
5. [Changelog - Legacy (380.x)](https://www.asuswrt-merlin.net/changelog-380) (External link)
6. [Changelog - Current (382.x and newer)](https://www.asuswrt-merlin.net/changelog) (External link)
7. [Installation](/RMerl/asuswrt-merlin.ng/wiki/Installation)
8. [Reverting](/RMerl/asuswrt-merlin.ng/wiki/Reverting/)

### Usage
1. [User scripts](/RMerl/asuswrt-merlin.ng/wiki/User-scripts)
2. [JFFS](/RMerl/asuswrt-merlin.ng/wiki/JFFS)
3. [Customizing configuration files](/RMerl/asuswrt-merlin.ng/wiki/Custom-config-files)
4. [DDNS Services](/RMerl/asuswrt-merlin.ng/wiki/DDNS-services)
5. [Custom DDNS support](/RMerl/asuswrt-merlin.ng/wiki/Custom-DDNS)
6. [SSHD](/RMerl/asuswrt-merlin.ng/wiki/SSHD)
7. [Scheduled tasks (Cron jobs)](/RMerl/asuswrt-merlin.ng/wiki/Scheduled-tasks-(cron-jobs))
8. [Enhanced traffic monitoring](/RMerl/asuswrt-merlin.ng/wiki/Enhanced-Traffic-monitoring)
9. [Adjustable TCP/IP connection tracking setting](/RMerl/asuswrt-merlin.ng/wiki/Adjustable-TCPIP-connection-tracking)
10. [Mounting remote CIFS shares](/RMerl/asuswrt-merlin.ng/wiki/Mounting-remote-CIFS-shares)
11. [Disk Spindown when idle](/RMerl/asuswrt-merlin.ng/wiki/Disk-Spindown-when-idle)
12. [NFS Exports](/RMerl/asuswrt-merlin.ng/wiki/NFS-Exports)
13. [DNS Director](/RMerl/asuswrt-merlin.ng/wiki/DNS-Director)
14. [Using a custom webui/FTP SSL certificate](/RMerl/asuswrt-merlin.ng/wiki/Custom-SSL-certificates)
15. [Wi-Fi Radar](/RMerl/asuswrt-merlin.ng/wiki/Wi-Fi-Radar)
16. [DNS Privacy](/RMerl/asuswrt-merlin.ng/wiki/DNS-Privacy) (DNS-over-TLS)
17. [AiMesh](/RMerl/asuswrt-merlin.ng/wiki/AiMesh)
18. [AMTM - Asuswrt-Merlin Terminal Menu](/RMerl/asuswrt-merlin.ng/wiki/AMTM)
19. [Policy-based routing through VPN Director](/RMerl/asuswrt-merlin.ng/wiki/VPN-Director)

### OpenVPN
1. [About OpenVPN](/RMerl/asuswrt-merlin.ng/wiki/About-OpenVPN)
2. [Setting up OpenVPN](/RMerl/asuswrt-merlin.ng/wiki/Configuring-OpenVPN)
3. [Generating certs with Easy-RSA](/RMerl/asuswrt-merlin.ng/wiki/Generating-OpenVPN-keys-using-Easy-RSA)
4. [Policy-based routing (Before version 386.3)](/RMerl/asuswrt-merlin.ng/wiki/Policy-based-routing)
5. [Policy-based routing - manual method **v380.xx firmware or later** now **DEPRECATED**](/RMerl/asuswrt-merlin.ng/wiki/Policy-based-routing-(manual-method))
6. [Policy-based Port (or MAC address) routing - manual method](/RMerl/asuswrt-merlin.ng/wiki/Policy-based-Port-routing-(manual-method))
7. [Static ip for OpenVPN clients](https://github.com/RMerl/asuswrt-merlin.ng/wiki/Static-ip-for-OpenVPN-clients)

### External software repositories

#### Entware
1. [Setting up Entware](/RMerl/asuswrt-merlin.ng/wiki/Entware) (Optware alternative)
2. [Setting up Entware](https://github.com/Entware-ng/Entware-ng/wiki/Install-on-asuswrt-merlin-firmware) (External link)
3. [Installing Transmission through Entware](/RMerl/asuswrt-merlin.ng/wiki/Installing-Transmission-through-Entware)
4. [Lighttpd web server with PHP support through Entware](/RMerl/asuswrt-merlin.ng/wiki/Lighttpd-web-server-with-PHP-support-through-Entware)
5. [Installing RTorrent through Entware](https://github.com/Entware-ng/Entware-ng/wiki/Using-Rtorrent) (External link)
6. [Installing Deluge through Entware](/RMerl/asuswrt-merlin.ng/wiki/Installing-Deluge-through-Entware)
7. [Webcam video surveillance Entware](http://www.hqt.ro/webcam-video-surveillance-via-mjpg-streamer-entware/) (External link)

#### Chroot Debian
1. [Plex Media Server on Arm Routers](http://www.hqt.ro/plex-media-server-through-debian-arm/) (External link, [Accessed by The Wayback Machine](https://web.archive.org/web/20230130003657/https://hqt.ro/plex-media-server-through-debian-arm/)) 
2. [Plex Media Server on Armhf (RT-AX58U etc.) Routers](https://web.archive.org/web/20230512030731/https://hqt.ro/plex-media-server-on-asuswrt-armhf-routers/) (External link, accessed by the The Wayback Machine)
3. [Debian 12 Bookworm + Plex Media Server (Asus RT-AX86S)](https://www.snbforums.com/threads/debian-12-bookworm-plex-media-server-asus-rt-ax86s.86175/) (External link)*
*the most up-to-date guide, written for Arm64 routers, but works for Armhf too, just change to your processor architecture when installing debian through debootstrap)
4. [Minidlna Upnp Media Server through debian](/RMerl/asuswrt-merlin.ng/wiki/Media-Server-through-debian) (link list)

### Development
1. [Download the latest source code from GitHub](/RMerl/asuswrt-merlin.ng/wiki/Download-the-latest-source-code-from-GitHub)
2. [Compile from source](https://github.com/RMerl/asuswrt-merlin.ng/wiki/Compile-Firmware-from-source)
3. Setting up a virtual machine for building:
   1. [Setting up a build VM under WSL2](/RMerl/asuswrt-merlin.ng/wiki/Setting-up-Build-VM-under-WSL2)
   2. [Setting up a build VM under Multipass](/RMerl/asuswrt-merlin.ng/wiki/Setting-up-Build-VM-under-Multipass)
   3. [Setting up a build VM in Docker](/RMerl/asuswrt-merlin.ng/wiki/Setting-up-Build-VM-in-Docker)
4. [Apply patches to source files](/RMerl/asuswrt-merlin.ng/wiki/Applying-patches-to-source-files)
5. [Addons API](/RMerl/asuswrt-merlin.ng/wiki/Addons-API)
6. **_OBSOLETE build instructions for older Linux versions_**:
    1. [Compile from source using Ubuntu](/RMerl/asuswrt-merlin.ng/wiki/OBSOLETE-Compile-Firmware-from-source-using-Ubuntu)
    2. [Compile from source using Linux Mint](/RMerl/asuswrt-merlin.ng/wiki/OBSOLETE-Compile-Firmware-from-source-using-Linux-Mint)
    3. [Compiling from source using a Debian-based Linux Distribution](/RMerl/asuswrt-merlin.ng/wiki/OBSOLETE-Compiling-Firmware-from-source-using-a-Debian-based-Linux-Distribution)

### Networking HowTo and Guides
1. [Iptables tricks and tips](/RMerl/asuswrt-merlin.ng/wiki/Iptables-tips)
2. [How to use Adblock Plus filter subscriptions to provide advertisement filtering to devices](/RMerl/asuswrt-merlin.ng/wiki/How-to-use-Adblock-Plus-filter-subscriptions-to-provide-advertisement-filtering-to-devices)
3. [Secure DNS queries using DNSCrypt](/RMerl/asuswrt-merlin.ng/wiki/Secure-DNS-queries-using-DNSCrypt)
4. [Setting up an IPv6 tunnel through Hurricane Electric](/RMerl/asuswrt-merlin.ng/wiki/IPv6-tunnelling)
5. [How to dedicate SSID for VPN and SSID for regular ISP using OpenVPN](/RMerl/asuswrt-merlin.ng/wiki/How-to-setup-SSID-for-VPN-and-SSID-for-Regular-ISP-using-OpenVPN.)
6. [How to use ipset to block connections](/RMerl/asuswrt-merlin.ng/wiki/Using-ipset)
7. [Link Aggregation Setup](/RMerl/asuswrt-merlin.ng/wiki/Link-Aggregation)
8. [Access modem Web UI on WAN port (no script)](/RMerl/asuswrt-merlin.ng/wiki/Access-modem-Web-UI-on-WAN-port-(no-script))
9. [Enforce the use of Google Safesearch on your LAN](/RMerl/asuswrt-merlin.ng/wiki/Enforce-Safesearch)
10. [How to have dedicated DHCP options bind to a specific SSID](/RMerl/asuswrt-merlin.ng/wiki/How-to-have-dedicated-DHCP-options-bind-to-a-specific-SSID)
11. [Custom domains with dnsmasq](/RMerl/asuswrt-merlin.ng/wiki/Custom-domains-with-dnsmasq)
12. [How to use Adblock using Pixelserv](https://github.com/RMerl/asuswrt-merlin.ng/wiki/How-to-use-Adblock-using-Pixelserv)
13. [How to block scanners, bots, malware, ransomware](https://github.com/RMerl/asuswrt-merlin.ng/wiki/How-to-block-scanners,-bots,-malware,-ransomware)
14. [Adaptive QoS Optimization](/RMerl/asuswrt-merlin.ng/wiki/Adaptive-QoS-Optimization)
15. [Installing Tailscale through Entware](/RMerl/asuswrt-merlin.ng/wiki/Installing-Tailscale-through-Entware)


### Misc HowTo and Guides
1. [Email notification from your router](/RMerl/asuswrt-merlin.ng/wiki/Sending-Email)
2. [WOL Script Wake Up Your Webserver On Internet Traffic](/RMerl/asuswrt-merlin.ng/wiki/WOL-Script-Wake-Up-Your-Webserver-On-Internet-Traffic)
3. [Scheduled LED control](/RMerl/asuswrt-merlin.ng/wiki/Scheduled-LED-control)
4. [How to make a NTFS usb hdd running more stable as media server, by ChrisR](/RMerl/asuswrt-merlin.ng/wiki/How-to--NTFS-usb-hdd-was-not-running-stable-as-media-server)
5. [Network Image Scanning With Sane](/RMerl/asuswrt-merlin.ng/wiki/Network-Scanning-With-Sane)
6. [Delay start of minidlna to wait for the USB disk mount](/RMerl/asuswrt-merlin.ng/wiki/delay-start-of-minidlna-to-wait-for-the-USB-disk-mount)
7. [Setting-up-FreeRadius2-through-Entware](/RMerl/asuswrt-merlin.ng/wiki/Setting-up-FreeRadius2-through-Entware)
8. [User NVRAM Save/Restore](/RMerl/asuswrt-merlin.ng/wiki/NVRAM-Save-Restore-Utility)
9. [Transfer (sync) a backup to a remote location using Rsync through a SSH tunnel between 2 Asus routers](/RMerl/asuswrt-merlin.ng/wiki/Transfer-(sync)-a-backup-to-a-remote-location-using-Rsync-through-a-SSH-tunnel-between-2-Asus-routers)
10. [Setting a random password for guest wifi](/RMerl/asuswrt-merlin.ng/wiki/Setting-a-random-password-for-guest-wifi)
11. [Guest WIFI QR code generator for display on local network webpage (visible from TV, smartphones...) and random password rotation](/RMerl/asuswrt-merlin.ng/wiki/Guest-WIFI-QR-code-generator-for-display-on-local-network-webpage-(visible-from-TV,-smartphones...)-and-random-password-rotation)
12. [Tinc VPN on AsusWRT-Merlin](http://nwgat.ninja/tinc-vpn-on-asuswrt-merlin/) (External Link)
13. [LUKS Encrypted USB Drive HOWTO](LUKS-Encrypted-USB-Drive-HOWTO)
14. [USB Disk Check at Boot](/RMerl/asuswrt-merlin.ng/wiki/USB-Disk-Check-at-Boot)
15. [USB Disk Check at Boot or Hot Plug (improved version)](https://github.com/RMerl/asuswrt-merlin.ng/wiki/USB-Disk-Check-at-Boot-or-Hot-Plug-(improved-version))
16. [Minidlna - Common Issues & Solutions](/RMerl/asuswrt-merlin.ng/wiki/Minidlna-‐-Common-Issues-&-Solutions)
17. [pyTivo - How-To Guide](/RMerl/asuswrt-merlin.ng/wiki/pyTivo-on-AsusWRT-Merlin-Router-‐-How-To-Guide)
18. [Setting up an NTP time server for your LAN](/RMerl/asuswrt-merlin.ng/wiki/Setting-up-an-NTP-time-server-for-your-LAN)
19. [Disk formatting](/RMerl/asuswrt-merlin.ng/wiki/Disk-formatting)
20. [Change the webui language](/RMerl/asuswrt-merlin.ng/wiki/Change-the-webui-language-when-it's-factory-locked)
21. [Restart WAN interface when internet is down](/RMerl/asuswrt-merlin.ng/wiki/Restart-WAN-interface-when-internet-is-down)
22. [Installing and running openSSH on Merlin](/RMerl/asuswrt-merlin.ng/wiki/Installing-and-running-openSSH-on-Merlin)
23. [Enable PXE booting into netboot.xyz](/RMerl/asuswrt-merlin.ng/wiki/Enable-PXE-booting-into-netboot.xyz)

### Reference:
1. [FAQ](/RMerl/asuswrt-merlin.ng/wiki/FAQ)
2. [Credits](/RMerl/asuswrt-merlin.ng/wiki/Credits/)
3. [Contact](/RMerl/asuswrt-merlin.ng/wiki/Contact/)
4. [Disclaimer](/RMerl/asuswrt-merlin.ng/wiki/Disclaimer/)
5. [Privacy](/RMerl/asuswrt-merlin.ng/wiki/Privacy-disclosure)