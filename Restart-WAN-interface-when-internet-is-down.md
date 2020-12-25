If you frequently lose internet connectivity with error like "Your ISP DHCP does not function correctly" then you might want these script. It is most useful for people with some router model like AC86U.

# Advanced script

Fantastic scripts have been written by community users. It is better to use them. See these links.

https://www.snbforums.com/threads/need-a-script-that-auto-reboot-if-internet-is-down.43819/post-371791

https://github.com/MartineauUK/Chk-WAN


# Simple script

Here below is a very basic script to run every 3 minutes and restart WAN interface if 10 pings fail. Adjust as desired. Please see above for more comprehensive scripts that do a better job.

1. Create and edit a new script file:
```
nano /jffs/scripts/custom_wan_monitor
```

2. Write in this script:
```
#!/bin/sh

PING_HOST=1.1.1.1
PING_WAIT=2
MAX_TRIES=10

wan_monitor(){
    echo "WAN Monitor: $@"|/usr/bin/logger -s
}
restart_wan_interface(){
    wan_monitor "Force restarting WAN interface now..."
    service "restart_wan_if 0"
    sleep 10
    service "restart_wan_if 1"
    wan_monitor "WAN interface is restarted."
}
ping_test(){
    count_tries=0
    ping_test_passed=0
    wan_monitor "Running ping test..."
    while [ $count_tries -lt $MAX_TRIES ]; do
        if /bin/ping -c 1 -W $PING_WAIT $@ >/tmp/wan_check.log; then
            ping_test_passed=1
            wan_monitor "Ping test succeeded within $PING_WAIT secs."
            break
        else
            sleep 1
            let count_tries=count_tries+1
            wan_monitor "Ping failed [$count_tries]"
        fi
    done
}

ping_test $PING_HOST

if [ $ping_test_passed -gt 0 ]; then
    wan_monitor "Internet was reachable. No need to restart WAN."
    break
else
    wan_monitor "Pings failed. Internet must be down."
    restart_wan_interface
fi
```

3. Run this full command:
```
touch /jffs/scripts/init-start ; echo 'cru a custom_wan_monitor "*/3 * * * * /jffs/scripts/custom_wan_monitor"' >> /jffs/scripts/init-start ; cat /jffs/scripts/init-start
```
See that it appended a new line to `init-start`.

4. Make both executable:

```
chmod 755 /jffs/scripts/init-start /jffs/scripts/custom_wan_monitor 
```

Reboot router.

See cron job is now listed:
```
cru l
```

Check syslog to see that script runs continuously.

Read here for help customizing cron job timing.
https://crontab.guru/

# References

Some relevant links for the ongoing issue:

https://www.snbforums.com/threads/your-isps-dhcp-does-not-function-correctly.43178/

https://www.snbforums.com/threads/new-rt-ax88u-no-wan-4-days-later.57960/

https://www.snbforums.com/threads/ac3100-primary-wan-assigned-show-your-isps-dhcp-does-not-function-correctly.61902/

https://www.snbforums.com/threads/asus-firmware-dhcp-continuous-mode-potential-fix-for-isps-dhcp-did-not-function-properly.61907/

https://www.snbforums.com/threads/your-isps-dhcp-does-not-function-correctly.67033/

https://www.snbforums.com/threads/rt-ax88u-router-behind-modem-router-isp.67312/

https://www.snbforums.com/threads/rt-ax88u-internet-status-disconnected.67331/

https://www.snbforums.com/threads/ac86u-random-disconnects-but-stay-on.67927/

https://www.snbforums.com/threads/ax88u-behind-isp-router-internet-drops-vpn-isp-dhcp-does-not-function.68438/

https://www.snbforums.com/threads/wan_connection-isps-dhcp-did-not-function-properly.56226/