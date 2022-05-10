### About VPN Director
VPN Director lets you create rules to determine how your LAN traffic is redirected through your OpenVPN clients.  You can create rules based on their local or remote IP (or even both).  Subnets in CIDR notation (i.e. 192.168.50.0/24) can also be used to cover IP ranges.

Introduced in version 386.3, this replaces the previous Policy Rules implementation by a global configuration where rules can be centrally managed instead of having to edit them for each individual client.

![VPN Director screenshot](https://www.asuswrt-merlin.net/sites/default/files/pictures/vpndirector.png)

People who had Policy Rules configured in previous firmware releases will automatically have these migrated to VPN Director.  You might need to review them as you might have some duplicate WAN entries if they existed on multiple clients.  Policy Rule and Policy Rule (strict) modes will automatically be switched to VPN Director mode.


### Rules Configuration
First, make sure you set the OpenVPN clients you want to affect into "VPN Director" routing mode.

On the VPN Director page you can create rules, by selecting the local and remote IPs (these are optional, leave one of the fields empty to indicate "any IP"), as well as the destination (either **WAN** to bypass all OpenVPN tunnels, or one of the five OpenVPN clients).   You can also enter a short description to identify that rule on the web interface.

On the main list, rules can be enabled/disabled by clicking on the icon in the leftmost column.

After making any change, don't forget to apply these changes by clicking on the Apply button at the bottom.  Rules will be immediately applied without having to restart running clients.

Up to a maximum of 199 rules can be created in total (that maximum can be lower if you use very long descriptions, as the total rules length must not exceed 7999 characters).


### Priority
Rules are applied in the following order:

* Rules with a WAN destination
* Rules with an OpenVPN 1 destination
* Rules with an OpenVPN 2 destination
* ...
* Rules with an OpenVPN 5 destination

Also note that any routes configured on the Dual WAN page will have a higher priority than all of these.


### Various notes

* You can start/stop clients on the VPN Director page by clicking on the stop/start link in the Status table.  (don't forget to Apply your changes first)
* Clients affected by DNS Exclusive mode are also prioritized (so if a client is affected by DNS Exclusive mode, it will be applied even if a later client rule would make it not Exclusive)
* Keep in mind that the killswitch will affect how lower priority clients work.  If a higher priority client triggers its killswitch, then no traffic will go through the lower priority clients.  You will generally want your lowest priority client to use the killswitch if you use multiple clients.
* VPN Director rules are stored in the /jffs partition rather than in nvram (due to their size), in the same directory that contains OpenVPN keys and certificates.
* In previous versions of the firmware before the VPN Director existed, Remote IP could be set to 0.0.0.0 for policy rules.  However, with VPN Director, the Remote IP should be left blank if the intention is to specify any or all IP addresses.  If not, there could be issues with the kill switch not behaving as expected.
