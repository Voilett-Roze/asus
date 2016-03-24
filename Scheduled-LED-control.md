Starting with 3.0.0.4.246.20, the router LEDs can now be controlled by the user.  The simplest way is to enable the Stealth Mode option (under Tools -> Other Settings), which will disable them.  It is also possible to turn them on and off on a specific schedule through cron jobs.

The following example will let you configure your router so that LEDs will turn themselves off at 18:00, and turn back on at 6:00.

First, make sure you have enabled the [JFFS](https://github.com/RMerl/asuswrt-merlin/wiki/JFFS) partition and support for custom user scripts.  Next, either create the following services-start script (or add to any existing script):

services-start:
```
#!/bin/sh
cru a lightsoff "0 18 * * * /jffs/scripts/ledsoff.sh"
cru a lightson "0 6 * * * /jffs/scripts/ledson.sh"
```
You can adjust the time at which you want both events to fire off by adjusting the second digit (which represents the hour).

Then, create the following two scripts, and also save them in /jffs/scripts/ :

ledsoff.sh:
```
#!/bin/sh
nvram set led_disable=1
service restart_leds
```

ledson.sh:
```
#!/bin/sh
nvram set led_disable=0
service restart_leds
```
Or use an all in one script save it to /jffs/scripts/ledcontrol this is intended replacement to ledsoff.sh and ledson.sh with ledcontrol this is for those that don't want to clutter with several files, usage is simple:

ledcontrol -on
ledcontrol -off
```
#!/bin/sh
show_help()
{
        echo "usage:"
        echo "ledcontrol -on         Turn leds on"
        echo "ledcontrol -off        Turn leds off"
        echo ""
}

option="${1}"
case ${option} in

                -on)
                    nvram set led_disable=0
                    service restart_leds
                    echo "Leds are now on"
                    logger "Leds: on"
                ;;

                -off)
                    nvram set led_disable=1
                    service restart_leds
                    echo "Leds are now off"
                    logger "Leds: off"
                ;;

                *) show_help
                exit 1
                ;;
esac
```
Then, reboot your router.  You can confirm that the jobs are properly set up by running the following command through telnet:

```
cru l
```

You should see both scheduled tasks listed.

If you want the current router state to be preserved between reboots, add the following commit call to both scripts (right after the nvram set):

```
nvram commit
```
Note however that this will cause additional wear on your flash, and might slightly shorten its life, as the flash RAM has a limited possible amount of write cycles.
