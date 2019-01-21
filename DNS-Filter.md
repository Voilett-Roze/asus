Under _LAN_ there is a tab called _DNSFilter_.  On this page you can force the use of a specific nameserver (DNS) that provides security/parental filtering.  This can be done globally, or on a per device basis.  Each of them can have a nameserver enforced.  For example, you can have your LAN use OpenDNS's server to provide basic filtering, but force your children's devices to use Yandex's family DNS server that filters out malicious and adult content.

If using a global filter, then specific devices can be told to bypass the global filter, by creating a client rule for these, and setting it to "_No Filtering_".

DNSFilter also lets you define up to three custom nameservers, for use in filtering rules (as Custom 1, 2 or 3). This will let you use any unsupported filtering nameserver.

You can configure a filter rule to force your clients to use whichever DNS is provided by the router's DHCP server (if you changed it from the default value, otherwise it will be the router's IP). Set the filtering rule to "_Router_" for this.

Note that DNSFilter will interfere with resolution of local hostnames.  This is a side effect of having devices forced to use a specific external nameserver. If this is an issue for you, then set the default filter to "_None_", and only filter out specific devices.