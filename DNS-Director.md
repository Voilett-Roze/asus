DNS Director (originally called DNSFilter, not to be confused with the company bearing the same name) is a feature that allows you to force specific devices on your network to use specific nameservers (DNS).  This can be done globally, or on a per device basis. Each of them can have a different nameserver enforced. For example, you can have your LAN use Quad9's server to provide basic filtering of malicious sites, but force your children's devices to use CleanBrowsing's family server that filters out both malicious and adult content.

The configuration can be found in the DNS Director tab, located in the LAN section:

![DNS Director](https://www.asuswrt-merlin.net/sites/default/files/pictures/DNS_Director.png)

If using a global redirection, then specific devices can be told to bypass the global redirection, by creating a client rule for these, and setting it to "No Redirection".

DNS Director also lets you define up to three custom nameservers, for use in redirection rules (as User-Defined 1, 2 or 3). This will let you use any unlisted nameserver.

You can configure a redirection rule to force your clients to use whichever DNS is provided by the router's DHCP server (if you changed it from the default value, otherwise it will be the router's IP). Set the filtering rule to "Router" for this.

Note that DNS Director will interfere with resolution of local hostnames. This is a side effect of having devices forced to use a specific external nameserver. If this is an issue for you, then set the default filter to "None", and only filter out specific devices.