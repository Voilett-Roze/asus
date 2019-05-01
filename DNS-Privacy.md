### DSN Privacy

Introduced in 384.11, DNS Privacy allows you to better secure your DNS queries through the use of a secured/encrypted connection.  At this time, only DNS-over-TLS (or DoT for short) is supported.

The feature can be enabled on the WAN -> Internet Connection page.  After enabling it, you must also enter at least one server in the DoT server list.  You can pick them from the Presets drop-down menu, or manually configure them in the list below.  Note that the server must explicitly support the DoT protocol, you cannot use a regular DNS there such as the ones provided by your ISP.

Description of the server entry fields:
* **Address:** The IP address of the DNS server.
* **TLS Port (optional):** Port to use (defaults to 853 if left blank).
* **TLS Hostname:** the TLS Hostname used by the security certificate.
* **SPKI Fingerprint (optional):** a SHA256 hash of the certificate (check if your server provider requires this, or leave empty)

To ensure enhanced security, it's recommended to also enable DNSSEC, and set it to also validate unsigned replies (make sure the DoT servers you use do support DNSSEC first, otherwise name resolution will fail).

> IMPORTANT: make sure you didn't enter a custom DNS server on the LAN -> DHCP page.  For DNS Privacy to work, the DHCP  server must point your client at the router's IP to use as their DNS server.  Likewise, your client must not be configured with a static DNS other than the router's IP.

You can monitor that DoT is properly being used by installing tcpdump through Entware, and monitoring trafic on ports 53 and 853 of the WAN interface (usually eth0):

```
tcpdump -i eth0 -p port 853 or 53 -n
```

Ideally, there should no longer be any trafic on port 53, and everything being sent to port 853.

> NOTE: There is currently an issue with the popular DoT/DoH [test site](https://cloudflare-dns.com/help/) provided by Cloudflare where it will fail to use properly signed DNSSEC hostnames during the test, causing the test to fail to correctly detect that you are using DoT.  This does not indicate that your setup doesn't work, and is something that will hopefully eventually be fixed by Cloudflare.  You can avoid this by temporarily disabling validation of unsigned records, however it is recommended to re-enable that option afterward.


## How this works
When enabled, the router will configure dnsmasq to instruct it to use the Stubby service running on the router instead of the DNS servers provided by your ISP (or manually configured in the _DNS Server 1_ and _2_ fields, if you have disabled the "_Connect to DNS Server automatically_" option).  Stubby will then send the name resolution queries to the configured DoT servers, and return their results to dnsmasq, which will apply DNSSEC validation, store the results in its cache, and provide the result to the requesting client.

Note that TLS requires a properly configured clock.  DNS Privacy won't use encryption until after your router's clock has been properly set through NTP.


### DNSFilter
DNSFilter still works as before, as an override for the DNS configuration used on your router's WAN page, or on the clients themselves.  If a client is configured to use a specific DNS server (for example, OpenDNS), then that client will still use OpenDNS, bypassing the DNS Privacy settings.  This also means you can force your clients to use DNS Privacy by configuring DNSFilter for "Router" mode.  This way, any DNS queries done on your LAN (even with hardcoded DNS servers, like the Netflix Android application which is hardcoded to use 8.8.8.8) will use DNS Privacy.


### OpenVPN Clients
This will mostly work as before.  OpenVPN clients with _"Accept DNS configuration"_ set to "Exclusive" will still use the DNS servers provided by the VPN server, bypassing DNS Privacy.  Setting DNS configuration to "Disabled" on the OpenVPN client configuration will allow it to use DNS Privacy, however note that some VPN providers will block the use of DNS servers other than their own, to protect you against leaking information by sending DNS queries outside of the tunnel.  If you trust the OpenVPN server you connect to, it's usually best to leave the setting to Exclusive mode - your DNS queries are already encrypted by the VPN tunnel anyway (for all clients configured to use the tunnel).
