Here is another tutorial about enabling dnscrypt on asuswrt routers.

[Install Entware](https://github.com/RMerl/asuswrt-merlin.ng/wiki/Entware#the-easy-way), then install necessary packages:

```
opkg install dnscrypt-proxy fake-hwclock
```

Tell router to use new resolver:
```
echo -e "#!/bin/sh\nsed -i '/^servers-file=.*/d' \$1" > /jffs/scripts/dnsmasq.postconf
echo "no-resolv" > /jffs/configs/dnsmasq.conf.add
echo "server=127.0.0.1#65053" >> /jffs/configs/dnsmasq.conf.add
```

start dnscrypt when router boots up
```
echo "/opt/etc/init.d/S09dnscrypt-proxy start" >> /jffs/scripts/services-start
```

set timezone variable for correct syslog times
```
echo "export TZ=$(cat /etc/TZ)" >> /opt/etc/profile
```

(optional) You can redirect using other DNS-servers on clients:
add to firewall-start or nat-start
```
iptables -t nat -A PREROUTING -i br0 -p udp --dport 53 -j DNAT --to $(nvram get lan_ipaddr)
iptables -t nat -A PREROUTING -i br0 -p tcp --dport 53 -j DNAT --to $(nvram get lan_ipaddr)
```

Reboot router to take effect:
```
reboot
```

Check if it is working

stop the dnscrypt service

```bash
/opt/etc/init.d/S09dnscrypt-proxy stop
```
Try to ping a url like

```bash
ping bing.com
```
The DNS resolution should not be working anymore
Turn it back on
```bash
/opt/etc/init.d/S09dnscrypt-proxy start
```

More info and discussion [here](http://www.snbforums.com/threads/dnscrypt-from-opendns.11645/).