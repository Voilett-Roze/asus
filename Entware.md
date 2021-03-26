### About
[Entware](http://github.com/Entware/entware) is a modern alternative to Optware.  Originally designed for OpenWRT, it is also usable by other firmware platforms such as DD-WRT or Tomato.  You can also set this up on your Asuswrt-Merlin based router.

For those unfamiliar with Optware: it's a software repository that offers various software programs that can be installed on your router.  They allow you to add new functionality to your router (provided you have the know-how to properly configure them).  This allows you, for example, to install and run the Asterisk SIP server on your router.

Note that you cannot use both Optware and Entware at the same time.

**Important:** Asus's DownloadMaster is based on Optware, and therefore is NOT compatible with Entware.  You will have to uninstall DownloadMaster and look at the alternatives provided by Entware.

After uninstalling, you should make sure "asusware.arm" or "asusware.*" dir on mounted disk partition is deleted. Otherwise, Entware won't work properly. After uninstalling DownloadMaster ensure the router is rebooted.


### Setup

You will need to plug a USB disk that's formatted in a native Linux filesystem (ext2, ext3 or ext4).

The installation and configuration process must be done through telnet or SSH.  If that part scares you, then forget about Entware already: everything must be installed and configured through telnet/SSH.

To start the installation process, first connect to your router over SSH.  
Then, launch the amtm application by simply running 
```
amtm
```
The menu will offer you an option to initiate the Entware installation.

If you are running a firmware version older than 384.15 (or 384.13_4 for the RT-AC87U and RT-AC3200), then you start the installation by running "entware-setup.sh" instead.


### Usage
Entware's management tool is called "opkg".  Just type "opkg" to see a list of available commands.  the most useful ones would be the following:

```
opkg list
opkg install software_name
opkg remove software_name
```

For example, to install nano, a text editor that is much more user-friendly than vi:

```
opkg install nano
```