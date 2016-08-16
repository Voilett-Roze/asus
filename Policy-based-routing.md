### Policy-based routing

When configuring your router to act as an OpenVPN client (for instance to connect your whole LAN to an OpenVPN tunnel provider), you can define policies that determines which clients, or which destinations should be routed through the tunnel, rather than having all of your traffic automatically routed through it.  This is also occasionally called "split-tunneling".

On the OpenVPN Clients page, set "_Redirect Internet traffic_" to "_Policy Rules_".  A new section will appear below, where you can add routing rules.  The "_Source IP_" is your local client (computer, mobile, etc...), while 
"_Destination_" is the remote server on the Internet.  The field can be left empty (or set to 0.0.0.0) to signify "_any IP_".  You can also specify a whole subnet, in CIDR notation (for example, _74.125.226.112/30_).

The "_Iface_" field (short for Interface) lets you determine if matching traffic should be sent through the VPN tunnel or through your regular Internet access (WAN).  This allows you to define exceptions (WAN rules being processed 
before the VPN rules).

You CANNOT configure a policy that will be based on a port through the webui - only on IPs (or subnets).  If you need more flexibility in your rules, you can look at this [alternate manual method](/RMerl/asuswrt-merlin/wiki/Policy-based-routing-(manual-method)).  Note that this method might interfere with other features, such as Adaptive QoS.

Also note that this feature is only compatible with OpenVPN tunnels using a TUN interface - it's not compatible with configurations set up with a TAP interface.


### Examples

To have all your clients use the VPN tunnel when trying to access an IP from this block that belongs to Google:

	RouteGoogle	0.0.0.0		74.125.0.0/16	VPN

Or, to have a computer routed through the tunnel except for requests sent to your ISP's SMTP server (assuming a fictious IP of 10.10.10.10 for your ISP's SMTP server):

	PC1		192.168.1.100	0.0.0.0		VPN
	PC1-bypass	192.168.1.100	10.10.10.10	WAN

Another setting exposed when enabling Policy routing is to prevent your routed clients from accessing the Internet if the VPN tunnel goes down.  To do so, enable "_Block routed clients if tunnel goes down_".
