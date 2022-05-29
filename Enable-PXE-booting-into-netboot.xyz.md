This will enable legacy BIOS, and UEFI devices to PXE boot into the https://github.com/netbootxyz/netboot.xyz menu.

Assume your AsusWRT-Merlin router is 192.168.1.1
1. LAN -> DHCP Server -> Basic Config: Set "Enable the DHCP Server" to Yes; IP Pool Starting Address: 192.168.1.2; IP Pool Ending Address: 192.168.1.254
2. Administration -> System -> Persistent JFFS2 partition: Set "Enable JFFS custom scripts and configs" to Yes
3. Administration -> System -> Service: "Enable SSH"
4. Reboot the box
5. `ssh username@192.168.1.1`
6. `cd /jffs`
7. `mkdir tftproot`
8. `cd tftproot`
9. `curl -o netboot.xyz.kpxe https://boot.netboot.xyz/ipxe/netboot.xyz.kpxe`
10. `curl -o netboot.xyz.efi https://boot.netboot.xyz/ipxe/netboot.xyz.efi`
11. `cd /jffs/configs`
12. `touch dnsmasq.conf.add`
13. `nano dnsmasq.conf.add` and add the following:

> enable-tftp  
> tftp-root=/jffs/tftproot  
> dhcp-match=set:bios,60,PXEClient:Arch:00000  
> dhcp-boot=tag:bios,netboot.xyz.kpxe,,192.168.1.1  
> dhcp-match=set:efi32,60,PXEClient:Arch:00002  
> dhcp-boot=tag:efi32,netboot.xyz.efi,,192.168.1.1  
> dhcp-match=set:efi32-1,60,PXEClient:Arch:00006  
> dhcp-boot=tag:efi32-1,netboot.xyz.efi,,192.168.1.1  
> dhcp-match=set:efi64,60,PXEClient:Arch:00007  
> dhcp-boot=tag:efi64,netboot.xyz.efi,,192.168.1.1  
> dhcp-match=set:efi64-1,60,PXEClient:Arch:00008  
> dhcp-boot=tag:efi64-1,netboot.xyz.efi,,192.168.1.1  
> dhcp-match=set:efi64-2,60,PXEClient:Arch:00009  
> dhcp-boot=tag:efi64-2,netboot.xyz.efi,,192.168.1.1  

14. `reboot` and wait until you can ping 192.168.1.1
15. from another device confirm that TFTP is working on the router

> `tftp 192.168.1.1`  
> tftp> `get netboot.xyz.kpxe`  
> Received 368475 bytes in 0.5 seconds  

16. Test with an UEFI device and with a legacy BIOS device that PXE booting is working (you might have enable PXE booting in the BIOS and/or in UEFI. For UEFI you usually have to enable UEFI Networking stack).


Frankenstein'd together from information found @
* https://programmingflow.com/2015/04/08/boot-any-machine-in-your-home-with-pxe.html
* https://netboot.xyz/docs/kb/networking/edgerouter/
* https://github.com/RMerl/asuswrt-merlin.ng/wiki/Custom-config-files