### About
QoS is a wonderful feature to let you optimize your network usage to prioritize the most important traffic on your network.  Having problems with Netflix stuttering when you're downloading updates?  Gaming pings suffering when your roommate is watching YouTube?  You need QoS.

There are 3 different methods of QoS available - Device priority, Traditional, and Adaptive.  This page is about the Adaptive QoS.

### Configuration

There are limitations in the stock Adaptive QoS provided by Asus, which has led to FreshJR creating his modification script.  For installation and configuration, use [AMTM](RMerl/asuswrt-merlin.ng/wiki/AMTM) and select the FreshJRQoS script.

One aspect of QoS configuration is which queueing discipline to use.  For more information on this, see [QoS Queue Disciplines](RMerl/asuswrt-merlin.ng/wiki/QoS-Queue-Disciplines)

In general, you want to use the following:
* no device priorities
* Adaptive QoS
* Manual bandwidth setting set to 85-95% of your actual speeds from a bandwidth test without QoS enabled
* fq_codel queue discipline

### Known Issues with Adaptive QoS

#### Adaptive QoS breaks DHCP renewal

If you are having problems with your WAN connection dropping or DHCP failing to renew, and you have Adaptive QoS enabled, you may be suffering from this problem.  Asus Adaptive QoS puts in place an iptables rule that blocks DHCP renewal when your lease expires.  For some reason new DHCP requests go through, but DHCP renewals do not.  If you are on a generous ISP which sets leases for days or weeks, you would not notice.  If you are on a less generous ISP like AT&T which sets DHCP lease for 10 minutes, your DHCP renewal fails and your WAN connection will drop every 10 minutes, all day long.

A known configuration that demonstrates this problem is AT&T FTTN / Uverse with Pace 5268ac gateway in DMZ+ mode to an Asus RT-AC68U router.  If you are experiencing this problem also, please update the list below to include your configuration.

##### Diagnosis

Check your system log for DHCP notices about DHCP renewals for your WAN IP address, e.g.

`dhcp_client: bound 108.228.12.xxx/255.255.252.0 via 108.228.12.1 for 600 seconds.`

or

`lldpd[309]: removal request for address of 108.228.12.xxx%4, but no knowledge of it`

To better see how your DHCP is behaving, make sure JFFS scripting is enabled, then SSH to your router and run these commands

`touch /jffs/scripts/wan-event
touch /jffs/scripts/dhcpc-event
chmod 755 /jffs/scripts/wan-event
chmod 755 /jffs/scripts/wan-event`

Then disable Adaptive QoS and watch your system log for information that your DHCP is renewing as expected, e.g.
`custom_script: Running /jffs/scripts/dhcpc-event (args: renew)`

Then re-enable your QoS and watch for your WAN being dropped and DHCP lease being acquired, e.g.

`custom_script: Running /jffs/scripts/dhcpc-event (args: deconfig)
custom_script: Running /jffs/scripts/wan-event (args: 0 disconnected)
custom_script: Running /jffs/scripts/wan-event (args: 0 stopped)
custom_script: Running /jffs/scripts/dhcpc-event (args: bound)
custom_script: Running /jffs/scripts/wan-event (args: 0 connected)`

The problem is a rule created by stock ASUS Adaptive QoS that is blocking DHCP traffic to your gateway.

##### Changing Iptables

If you think you have this problem, you can change the iptables rules to allow DHCP to your gateway by SSHing to your router and entering this command, changing "YOUR_GATEWAY_IP_ADDRESS" to the IP address of your gateway, e.g. 192.168.1.254 is the default IP for the Pace 5268ac gateway:

`iptables -t mangle -I PREROUTING 1 -i eth0 -s YOUR_GATEWAY_IP_ADDRESS/32 -p udp -m udp --sport 67 --dport 68 -j ACCEPT`

Then restart Adaptive QoS and see if your DHCP requests are getting through.

If not, you can recover by turning off Adaptive QoS and restarting your firewall with the command "service firewall_restart"

If it fixed your problem, you can make the solution permanent by adding the following to your /jffs/scripts/firewall-start script.  If that script does not exist you can create it with the touch and chmod commands above and substituting "firewall-start" for the filename.

`if [ "$(nvram get qos_enable)" = "1" ] && [ "$(nvram get qos_type)" = "1" ]; then
  iptables -t mangle -I PREROUTING 1 -i eth0 -s YOUR_GATEWAY_IP_ADDRESS/32 -p udp -m udp --sport 67 --dport 68 -j ACCEPT
fi`

then restart your router and verify QoS and DHCP are working as expected.

Once everything is working, you can remove the custom scripts created above with the commands

`rm /jffs/scripts/wan-event
rm /jffs/scripts/dhcpc-even`

##### Known Problem Configurations
* AC68U with Pace 5268ac gateway in DMZ+ mode
