To use your router as an NTP time server enable the `Enable local NTP server` option in Administration - System > Basic Config.

To intercept NTP connections from LAN clients and redirect them to the router's NTP server enable `Intercept NTP client requests`. Note that IPv6 is not supported.

Here is a simple guide for syncing your time from your router instead of from the internet

# 3rd Party Addons

There is a addon package for Asus-Merlin Firmware called ntpMerlin. ntpMerlin implements an NTP time server for AsusWRT Merlin with charts for daily, weekly and monthly summaries of performance. A choice between ntpd and chrony is available.
