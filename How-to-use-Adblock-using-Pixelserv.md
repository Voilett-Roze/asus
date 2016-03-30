# Foreword
Pixelserv is a tool used to create a 1x1 transparent image can be used in combination with an Adblocker (see notes) to remove ads without having a Adblocker extension in the browser and it works across your network with any device that is connected to your router.

## Pixelserv Installation Process
* Follow the official installation steps at: https://github.com/kvic-z/pixelserv-tls
* Or install from Entware via OPKG: `opkg install pixelserv-tls`
* For community support use this thread [pixelserv - A Better One-pixel Webserver for Adblock](http://www.snbforums.com/threads/pixelserv-a-better-one-pixel-webserver-for-adblock.26114/)
* Use a blocklists: [uBlockr](https://gitlab.com/spitfire-project/ublockr), [AB-Solution](https://github.com/decoderman/AB-Solution)

### Setting arguments in /opt/etc/init.d/S80pixelserv-tls

**Example:** _ARGS="`192.168.1.1 -p 80 -p 8080 -k 443 -k 2443 -u root"_<br/>
Makes pixelserv run on 192.168.1.1 at port 80,8080 for http and https 443,2443 as root

for other settings 
```
        -2 (disable HTTP 204 reply to generate_204 URLs)
        -f (stay in foreground - don't daemonize)
        -k https_port (443 if omitted)
        -l (log access to syslog)
        -n i/f (all interfaces if omitted)
        -o select_timeout (10 seconds)
        -p http_port (80 if omitted)
        -r (deprecated - ignored)
        -R (disable redirect to encoded path in tracker links)
        -s /relative_stats_html_URL (/servstats if omitted)
        -t /relative_stats_txt_URL (/servstats.txt if omitted)
        -u user ("nobody" if omitted)
        -z path_to_https_certs (/opt/var/cache/pixelserv if omitted)
```