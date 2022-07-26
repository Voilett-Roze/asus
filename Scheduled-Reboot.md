The following user script will make your router reboot itself every night at 4 am:

```
#!/bin/sh
cru a ScheduledReboot "0 4 * * * /sbin/reboot"
```

Put this inside an `/jffs/scripts/init-start` [user script](https://github.com/RMerl/asuswrt-merlin.ng/wiki/User-scripts).

From @krader1961: My question is why in the world would you schedule periodic reboots of a router? The only time I reboot my ASUS router is when there is a power outage or I have made a change to its configuration. If you're proposing scheduling a periodic reboot to deal with a bug you should say so and link to a description of the bug.