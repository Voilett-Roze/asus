## Introduction
Tailscale is a VPN service that makes the devices and applications you own accessible anywhere in the world, securely and effortlessly. It enables encrypted point-to-point connections using the open source [WireGuard](https://www.wireguard.com/) protocol, which means only devices on your private network can communicate with each other.

https://tailscale.com/kb/1151/what-is-tailscale

## Prerequisites
This installation guide assumes you are already familiar with the general operation of tailscale. Extensive documentation is available on the tailscale website and it is not the purpose of this guide to replicate it here. https://tailscale.com/kb/1017/install

You must already have a working installation on [Entware](https://github.com/RMerl/asuswrt-merlin.ng/wiki/Entware#the-easy-way) on your router.

You also need to have created a Tailscale account to use with your devices. https://tailscale.com/kb/1017/install#step-1-sign-up-for-an-account
## Installation
### 1. Install tailscale

Install the tailscale Entware package.
```
# opkg install tailscale
Installing tailscale (1.58.2-1) to root...
Downloading https://bin.entware.net/aarch64-k3.10/tailscale_1.58.2-1_aarch64-3.10.ipk
Installing ca-bundle (20230311-1) to root...
Downloading https://bin.entware.net/aarch64-k3.10/ca-bundle_20230311-1_all.ipk
Configuring ca-bundle.
Configuring tailscale.
```
### 2. Kernel vs. Userspace Mode
By default tailscale operates in [kernel mode](https://tailscale.com/kb/1177/kernel-vs-userspace-routers?q=userspace-networking). Kernel mode is more performant than userspace mode but may be incompatible with certain Merlin addons and features. Depending on which mode you choose the following changes are required.

**2.1 Kernel Mode**

Add/change the following lines in `/opt/etc/init.d/S06tailscaled`.
```
PRECMD="modprobe tun"
PREARGS="nohup"
```
Create a `/jffs/scripts/firewall-start` [user script](https://github.com/RMerl/asuswrt-merlin.ng/wiki/User-scripts) as follows, or add the line to the existing script if you have one. Remember to make the script executable.
```
#!/bin/sh
if [ -x /opt/bin/tailscale ]; then tailscale down; tailscale up; fi
```

**2.2 Userspace Mode**

Change the following lines in `/opt/etc/init.d/S06tailscaled`.
```
ARGS="--tun=userspace-networking --state=/opt/var/tailscaled.state"
PREARGS="nohup"
```
No firewall changes are needed in userspace mode.
### 3. Start tailscale
Use the following command to start the tailscale daemon, or reboot the router.
```
# /opt/etc/init.d/S06tailscaled start
 Starting tailscaled...              done.
```
### 4. Configure tailscale as an exit node and subnet router
This is a one-time task. If your LAN uses a different IP address or subnet mask change the command below to match your system.

Copy your unique authentication URL and paste it into a browser. Connect the device using your tailscale account. Find your router's name in the Tailscale Machines list, click on `...`/`Edit route settings...` and enable Subnet routes and Exit node.

Return to your terminal session and you should see the "Success" message.

```
# tailscale up --advertise-exit-node --advertise-routes=192.168.50.0/24

To authenticate, visit:

        https://login.tailscale.com/a/xxxxxxxxxxxxx

Success.
```

## Reconfiguring Tailscale
Tailscale's behaviour can be changed by using `tailscale up` to set or reset various flags. See the official documentation here: https://tailscale.com/kb/1241/tailscale-up

Examples:

To configure the router as a standalone device and stop advertising itself as an exit node and subnet router:
```
tailscale up --reset
```
To configure the router as an exit node but not a subnet router:
```
tailscale up --advertise-exit-node --reset
```
Many other options are available including LAN to LAN setups but that is beyond the scope of this wiki.

## Updating tailscale (or not).
I recommend you _don't_ update tailscale directly from the tailscale website using the commands below as it may not be fully compatible with asuswrt-merlin. The tailscale Entware package is periodically updated and can be upgraded just like any other Entware package using `opkg upgrade`.
```
# tailscale update
This will update Tailscale from 1.58 to 1.64.0. Continue? [y/n] y
Downloading "https://pkgs.tailscale.com/stable/tailscale_1.64.0_arm64.tgz"
Download size: 25751613
Downloaded 8720/25751613 (0.0%)
Downloaded 25751613/25751613 (100.0%)
Downloading "https://pkgs.tailscale.com/stable/tailscale_1.64.0_arm64.tgz.sig"
Signature OK
Extracting "/root/.cache/tailscale-update/tailscale_1.64.0_arm64.tgz"
Updated /tmp/mnt/TOSHIBA1/entware/bin/tailscale
Updated /tmp/mnt/TOSHIBA1/entware/bin/tailscaled
Tailscale binaries updated successfully.
Please restart tailscaled to finish the update.

# /opt/etc/init.d/S06tailscaled restart
 Shutting down tailscaled...              done.
 Starting tailscaled...              done.
```
## Uninstalling tailscale
To remove tailscale from your router run the following commands. If you made changes to `/jffs/scripts/firewall-start` you should undo that also.
```
# /opt/etc/init.d/S06tailscaled stop
 Checking tailscaled...              alive.
 Shutting down tailscaled...              done.
# opkg remove tailscale
Removing package tailscale from root...
# rm /opt/var/tailscaled.state
```
Now go to the tailscale [admin console](https://login.tailscale.com/admin/machines) and find the entry for the router. Click on `...`/`Remove...` and remove the machine.

## Known Issues
1. Be aware that upgrading the tailscale Entware package (`opkg upgrade`) will overwrite any changes made to `/opt/etc/init.d/S06tailscaled`. Make sure you know what changes you have made so that you can reapply them after the upgrade.
