# Setting up the Prosody chat server
The lightweight [XMPP](https://en.wikipedia.org/wiki/XMPP) chat server [Prosody](https://prosody.im) is available via [Entware](Entware).
```sh
opkg install prosody
```
The configuration and the data resides in `/opt/etc/prosody`.

To keep things simple, we will run the service using the predefined user and group `nas`. The end of `/opt/etc/prosody/prosody.cfg.lua` should be edited to something like the following:
```lua
log = {
	info = "/opt/var/log/prosody/prosody.log"; -- Change 'info' to 'debug' for verbose logging
	error = "/opt/var/log/prosody/prosody.err";
}
pidfile = "/opt/var/run/prosody/prosody.pid"
prosody_user = "nas"
prosody_group = "nas"
VirtualHost "myhostname.example.com"
	ssl = {
		certificate = "/opt/etc/prosody/certs/myhostname.example.com/fullchain.pem";
		key = "/opt/etc/prosody/certs/myhostname.example.com/privkey.pem";
	}
Component "conference.myhostname.example.com" "muc"
```
Replace each occurrence of `myhostname.example.com` with your fully qualified domain name.

Make sure that the directories exist and are writable by the user:
```sh
mkdir -m 750 /opt/etc/prosody/certs
mkdir /opt/var/log/prosody /opt/var/run/prosody
chown -R nas.nas /opt/etc/prosody /opt/var/log/prosody /opt/var/run/prosody
```
# Creating or renewing an SSL certificate
XMPP clients should refuse to connect to a server that lacks a certificate that is signed by a trusted certificate authority. Some trusted services offer to sign certificates free of charge. The following assumes that you are familiar with [LetsEncrypt](LetsEncrypt).
```sh
FQDN=myhostname.example.com
./acme.sh --certhome /opt/etc/prosody/certs --fullchain-file /opt/etc/prosody/certs/$FQDN/fullchain.pem --key-file /opt/etc/prosody/certs/$FQDN/privkey.pem --issue -d $FQDN --server letsencrypt --standalone
chown -R nas.nas /opt/etc/prosody/certs
prosodyctl reload
```
Replace `myhostname.example.com` or `$FQDN` with your fully qualified domain name.
# Starting up Prosody
Prosody will be automatically started when the router starts up, via `/opt/etc/init.d`.

If you are starting up Prosody for the first time without restarting the router, `prosodyctl start` should work.
# Creating user accounts
```sh
prosodyctl adduser username@myhostname.example.com
```
The command will ask for a password for the user. In XMPP clients, such as [Gajim](https://gajim.org), [Pidgin](https://pidgin.im), or [conversations.im](https://conversations.im) (also available via [F-Droid](https://f-droid.org)), you would enter `username@myhostname.example.com` as the user name.

# Opening a hole in the firewall
If the firewall is enabled in the ASUS web user interface, it will block connections from the WAN to the XMPP service. For example, an Android device would connect fine via the router-provided WLAN, but the connection attempt would seem to hang when using mobile data.

To enable connections from the WAN, create a [user script](User-scripts) `/jffs/scripts/firewall-start` with the following contents:
```sh
#!/bin/sh
iptables -I INPUT -p tcp -m tcp -i "$1" --dport 5222 --jump ACCEPT
iptables -I INPUT -p tcp -m tcp -i "$1" --dport 5269 --jump ACCEPT
```
This script will be run each time when **Enable Firewall** is changed to _Yes_ in the web user interface, or the router is started up.

The port number 5222 is for client-to-server connections and 5269 for server-to-server XMPP (s2s, federated network).