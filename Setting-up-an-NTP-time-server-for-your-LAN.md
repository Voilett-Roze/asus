To use your router as an NTP time server set the `Enable local NTP server` option to `Yes` in **Administration - System > Basic Config**.

To intercept NTP connections from LAN clients and redirect them to the router's NTP server enable `Intercept NTP client requests`. Note that IPv6 is not supported.

### 3rd Party Addons

There is an addon package for Asuswrt-Merlin Firmware called ntpMerlin. ntpMerlin implements an NTP time server for AsusWRT Merlin with charts for daily, weekly and monthly summaries of performance. A choice between ntpd and chrony is available.

https://www.snbforums.com/threads/ntpmerlin-v3-x.68508/