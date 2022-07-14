[Let's Encrypt](https://letsencrypt.org) is a non-profit certificate authority (CA) that can provide free-of-charge SSL certificates for services that require it.

# Let's Encrypt in the web UI
In the **WAN - DDNS** configuration tab in the ASUS web user interface, it appears to be possible to have the router manage LetsEncrypt certificates, if a dynamic DNS service has been set up.

The choice **Server: Custom** will use a [user script](User-scripts) `/jffs/scripts/ddns-start`. If your IP address is changing very seldomly, you might use a dummy script like this:
```sh
#!/bin/sh
exec /sbin/ddns_custom_updated 1
```
Unfortunately, Let's Encrypt would work in this way, at least not in the 386.7 release. It appeared that the `/usr/sbin/acme.sh` will not be invoked at all. In the 386.7 release, the script is based on version 2.8.3 of [acme.sh](https://github.com/acmesh-official/acme.sh/), which could be too old to work.

# Invoking Let's Encrypt via SSH
The script `acme.sh` supports a `--standalone` mode that will start a HTTP server in port 80 to implement a challenge-response connection with the certificate authority. This mode requires that port 80 is available. By default, the ASUS web server (`httpd`) will listen at port 80, even if HTTP logins are disabled in the web UI.

1. In the web UI, under **Administration - System** and **Local Access Config**, ensure that the **HTTP LAN port** is _not_ 80.
1. Invoke the following commands.
```sh
wget https://raw.githubusercontent.com/acmesh-official/acme.sh/master/acme.sh
chmod 700 acme.sh
./acme.sh --issue -d myhostname.example.com --server letsencrypt --standalone
```
## Notes
1. This will download the latest version of the script. As of this writing, it was a few commits ahead of the 3.0.4 release. As always, **you should be careful about downloading something from the Internet and executing it as the super user**.
1. The above commands may depend on something that was installed as part of [Entware](Entware).
1. Replace `myhostname.example.com` with your fully qualified domain name.
1. Since version 3.0.0, `acme.sh` defaults to ZeroSSL, which requires that an email address be provided. With `--server letsencrypt`, there is no need to specify any email address.