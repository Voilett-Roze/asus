### Policy-based routing

When configuring your router to act as an OpenVPN client (for instance to connect your whole LAN to an OpenVPN tunnel provider), you can define policies that determines which clients, or which destinations should be routed through the tunnel, rather than having all of your traffic automatically routed through it.  This is also occasionally called "split-tunneling".

On the OpenVPN Clients page, set "_Redirect Internet traffic_" to either "_Policy Rules_" or "_Policy Rules (strict)_".  Strict mode will take additional steps to ensure that there aren't any extra routes that could potentially bypass your tunnel, by only allowing routes that specifically target the tunnel's network interface.  This is usually preferred, however this will interfere with any route you might have manually configured on your WAN interface, which is why it is a separate option.

Once you enable Policy Rules, a new section will appear below, where you can add routing rules.  The "_Source IP_" is your local client (computer, mobile, etc...), while "_Destination_" is the remote server on the Internet.  The field can be left empty (or set to 0.0.0.0) to signify "_any IP_".  You can also specify a whole subnet, in CIDR notation (for example, _74.125.226.112/30_).

The "_Iface_" field (short for Interface) lets you determine if matching traffic should be sent through the VPN tunnel or through your regular Internet access (WAN).  This allows you to define exceptions (WAN rules being processed 
before the VPN rules).

By default, all traffic go through the WAN.  What you define there with a VPN _iface_ will be routed through the VPN.  Use the WAN _iface_ to configure exceptions to configured VPN rules (for instance, if you configure a /24 to be routed through the VPN, but want one IP within that /24 to be routed through the WAN instead).

You CANNOT configure a policy that will be based on a port through the webui - only on IPs (or subnets).  If you need more flexibility in your rules, you can look at this [alternate manual method](/RMerl/asuswrt-merlin.ng/wiki/Policy-based-routing-(manual-method)).  Note that this method might interfere with other features, such as Adaptive QoS.

Another setting exposed when enabling Policy routing is to prevent your routed clients from accessing the Internet if the VPN tunnel goes down.  To do so, enable "_Block routed clients if tunnel goes down_".

Also note that this feature is only compatible with OpenVPN tunnels using a TUN interface - it's not compatible with configurations set up with a TAP interface.


### DNS behaviour

For reference, the definition of the Accept DNS Configuration field values are as follows:

* Disabled: DNS servers pushed by VPN provided DNS server are ignored.
* Relaxed: DNS servers pushed by VPN provided DNS server are prepended to the current list of DNS servers, of which any can be used.
* Strict: DNS servers pushed by the VPN provided DNS server are prepended to the current list of DNS servers, which are used in order. Existing DNS servers are only used if VPN provided ones don’t respond.
* Exclusive: Only the pushed VPN provided DNS servers are used.

"_Accept DNS configuration_" to _Exclusive_, combined with Policy based routing, means that all clients that are configured to go through the VPN will use the DNS servers provided by the VPN tunnel, but those configured to go through the WAN will keep using the ISP's DNS. The disadvantage of setting “_Accept DNS configuration_” to “_Exclusive_” is that **dnsmasq** will be bypassed since the VPN tunnel will exclusively use the DNS of the VPN Provider. The popular [Diversion](https://diversion.ch/) ad blocker program, written for the Asuswrt-Merlin firmware, will not work since Diversion requires the features of **dnsmasq**. Diversion will work over the VPN tunnel when “Accept DNS configuration” is set to “Exclusive” and Policy Rules are disabled by setting “Redirect Internet Traffic” to “All”.

There are two options available if you want the OpenVPN client to use **dnsmasq** when using Policy Rules. This is done by setting "_Accept DNS Configuration_" to either "_Relaxed_,  "_Strict_" or "_Disabled_". The disadvantage is this will cause the DNS to "leak" for clients assigned to the VPN Clent. A work around solution is to use the built in DNS-over-TLS on the WAN page to encrypt DNS queries. 

Note that if there are multiple rules for a given client's IP (for instance if it has one rule stating that all its traffic is to go through the VPN, and an exception rule stating that traffic for a specific destination IP is to be kept through the WAN), all of its name resolution will still go through the VPN server's specified DNS.  This is because the router has no way of knowing if the DNS query is related to a specific destination.  Therefore, the safest behaviour gets used, and all the queries done by that client will use the VPN server's DNS.


### Examples

To have all your clients use the VPN tunnel when trying to access an IP from this block that belongs to Google:

	RouteGoogle	0.0.0.0		74.125.0.0/16	VPN

Or, to have a computer routed through the tunnel except for requests sent to your ISP's SMTP server (assuming a fictious IP of 10.10.10.10 for your ISP's SMTP server):

	PC1		192.168.1.100	0.0.0.0		VPN
	PC1-bypass	192.168.1.100	10.10.10.10	WAN

A common configuration setup where you want your whole LAN to go through the VPN, but not the router itself:

	LAN		192.168.1.0/24	0.0.0.0		VPN
	Router		192.168.1.1	0.0.0.0		WAN

NOTE: [Small Net Builder member @Xentrk](https://www.snbforums.com/members/xentrk.49161/) makes use of the IPSET technique for selective routing and includes additional selective routing features for LAN Clients, OpenVPN Clients and OpenVPN Servers. Refer to the [x3mRouting ~ Selective Routing for Asuswrt-Merlin Firmware](https://github.com/Xentrk/x3mRouting) on GitHub for more information.