## About user scripts
While Asuswrt-Merlin only adds a limited number of new features over the original firmware, a lot of customizations can be achieved through the use of user scripts.  These will allow you to set up custom firewall rules, create jobs that can be run at scheduled intervals, or start new services.

Those scripts are stored in the internal non-volatile flash in the [JFFS](https://github.com/RMerl/asuswrt-merlin.ng/wiki/JFFS) partition.  Support for these scripts must be enabled, under Administration -> System on the webui.

To enable user scripts on an aimesh node, SSH into the node and run the following commands:

```
nvram set jffs2_scripts="1"
nvram commit
reboot
```

## Available scripts:

These shell scripts will be run when certain events occur.  They must be saved in `/jffs/scripts/`.

These scripts will usually run in parallel to the event itself, except when explicitly documented as being a blocking script.  In that case, the script will prevent the execution of the event itself until either the script completes, or it times out (current default timeout value is 120 seconds).

### services-start
Called after all other system services have been started at boot.  This is the best place to stop one of these services and restart it with a different configuration, for example. (But be aware that anytime the service gets manually restarted it will revert back to the original setup.)

### services-stop
Called before all system services are stopped, usually on a reboot.

### service-event
Called before a service event is called (e.g. restart_httpd, restart_wireless, etc...).  First argument is the event (typically _stop_, _start_ or _restart_), second argument is the target (_wireless_, _httpd_, etc...).  This is a blocking script, meaning that it will prevent the execution of the event itself until the script either completes or times out.

### service-event-end
Introduced in firmware 384.11.  Called after a service event completes (e.g. restart_httpd, restart_wireless, etc...).  First argument is the event (typically _stop_, _start_ or _restart_), second argument is the target (_wireless_, _httpd_, etc...). 

### wan-event
Called after an event related to the WAN interface occurs.  The script will receive two parameters:

$1 will contain the WAN unit (0 for Primary, or 1 for Secondary).  
$2 will contain the event type, from the following list (the list can vary between firmware versions):
* init
* connecting
* connected
* disconnected
* stopped
* disabled
* stopping

The old wan-start script would occur on a "connected" event.  Note that after a connected event occurs, the Internet connection might not yet be fully functional.  You should probably add a slight pause before you try to initiate anything requiring Internet access, or write your own code to wait until the connection becomes functional.

### wan-start
_(deprecated since 384.15, please use wan-event instead)_

Called after the WAN interface came up.  Good place to put scripts that depend on the WAN interface (e.g. to update an IPv6 tunnel or a dynamic DNS service).  The Internet connection is unlikely to be active when this script is run.  Add a `sleep` line to delay running until the connection is complete, or loop until your command succeeds.

### firewall-start
Called after the firewall got started and filtering rules have been applied.  This is where you will usually put your own custom rules in the filter table (but _not_ the NAT table).  The script receives the WAN interface name (e.g. `ppp0`) as an argument which can be used in the script using `$1`.

### nat-start
Called after NAT rules (i.e. port forwards and such) have been applied to the NAT table.  This is where you will want to put your own NAT table custom rules (e.g. a port forward that only allows connections coming from a specific IP).

### init-start
Called right after JFFS got mounted, and before any of the services get started. This is the earliest part of the boot process where you can insert something.

### pre-mount
Called just before a partition gets mounted.  This is run in a blocking call and will block the mounting of the partition for which it is invoked until its execution is complete or it times out.  This is done so that it can be used for things like running `e2fsck` on the partition before mounting.  This script is passed the device path being mounted (e.g. `/dev/sda1`) as an argument which can be used in the script using `$1`. Since firmware 384.11 a second argument is passed (`$2`) that contains the filesystem type (e.g. `ext3`).

### post-mount
Called just after a partition got mounted.  The script is passed the mount point (the filesystem path where the partition was mounted, e.g. `/tmp/mnt/OPT`) as an argument which can be used in the script using `$1`.

### unmount
Called just before unmounting a partition.  Like pre-mount, this is a blocking script, so be careful with it.  The script is passed the mount point as an argument which can be used in the script using `$1`.

### dhcpc-event
Called whenever a DHCP event occurs on the WAN interface.  The type of event is passed as an argument which can be used in the script using `$1`; possible event types in the version of `udhcpc` in ASUSWRT are `deconfig` (when udhcpc starts and when a lease is lost), `bound` (when a lease and new IP address are acquired), and `renew` (when a lease is renewed, but the IP did not change).

### openvpn-event
Called whenever an OpenVPN server gets started/stopped, or an OpenVPN client connects to a remote server.  Uses the same syntax/parameters as the "up" and "down" scripts in OpenVPN. The `$script_type` environment variable indicates what sort of event occurred on the device `$dev`.

### ddns-start
Called at the end of a DDNS update process.  This script is also called when setting the DDNS type to "Custom".  The script gets passed the WAN IP as an argument, which can be used in the script using `$1`.  When handling a "Custom" DDNS, this script is also responsible for reporting the success or failure of the update process.  See the [Custom DDNS](https://github.com/RMerl/asuswrt-merlin.ng/wiki/Custom-DDNS) section for more information.

### update-notification
Called when the scheduled new firmware version availability check detects there's a new firmware available for download. See [update notification example](https://github.com/RMerl/asuswrt-merlin.ng/wiki/update-notification-example) for more info.

### qos-start
Called when Traditional QOS or Cake are done creating their stop/start script, and before running it.  You can use this script to modify the /etc/cake-qos.conf config file (contains variables used by Cake) or /tmp/qos (generated for both modes, to create/remove the tc qdisc/rule entries).  This script gets one argument passed as parameter, which will contain "init" (when about to start QOS) or "rules" (when iptables rules have been configured for Traditional QOS).  This script is run in a blocking call and will pause starting QOS until it completes.

### wgserver-stop and wgserver-start
Called when the WireGuard server is stopped or started.

### wgclient-stop and wgclient-start
Called when a WireGuard client is stopped or started.  The client unit number (1-5) is passed to the script as argument.


## Postconf scripts
Note that in addition to these, you can also use the numerous postconf scripts supported by the firmware, which allow you to execute a script between the moment a service's config file is generated and the service is about to be executed.  See the [Postconf scripts](https://github.com/RMerl/asuswrt-merlin.ng/wiki/Custom-config-files#postconf-scripts) section for more information.


## Creating scripts
Don't forget to set any script you create as being executable:

```
chmod a+rx /jffs/scripts/*
```

And like any UNIX script, they need to start with a shebang:

```
#!/bin/sh
```

Also, you must save files with UNIX line endings.  Note that Windows's Notepad cannot save with UNIX line endings; use Notepad++ instead.  You can also directly edit them on the router through SSH, by using `vi` or `nano`, both included in the firmware; they will create files with the proper line endings.


## Troubleshooting scripts:
Try running your script manually at first to make sure there is no syntax error in it.  You can also insert some code near the top to be able to easily determine whether or not your script ran.  For example:

```
touch /tmp/000wanstarted
```

You can then easily tell that the script did run by looking for the presence of 000wanstarted in the `/tmp` directory.

A useful command for debugging user scripts is `logger`, which will log messages to the system log, visible in the Web UI.