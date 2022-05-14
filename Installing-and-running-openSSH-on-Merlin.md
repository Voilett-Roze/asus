#### Installing and running openSSH on Merlin

I got tired of dropbear taking too long to establish a connection, and then dropping the connection every minute or two. So I got an openssh-server running on Merlin.

* Tested on RT-AC68U, running 386.5_2
* I'm documenting sort of as go. This was not thoroughly tested from scratch, and my documentation may be incomplete or wrong, but it should still serve as a starting point; hopefully others can confirm that it works, or update it as needed
* I'm assuming that you know and understand the basics of setting up and configuring an ssh server and client via CLI
* I'm glossing over or completely ignoring prerequisites (eg opkg, jffs, how to edit files)
* I'm glossing over configuration and setup options (eg RSA vs DSA vs ECDSA) which may or may not be more or less suitable to other environments, or even my own setup
* For simplicity, these examples assume that sshd should only be listing on the LAN, and that's on 192.168.1.1
* There are probably things that can be done better. I'm here to learn.

#### Get started

Install the openssh packages: `opkg install openssh-keygen openssh-server`


Generate a server key: `ssh-keygen -f /opt/etc/ssh/ssh_host_rsa_key`

If you have one, put a public key into `~/.ssh/authorized_keys`

Edit `/opt/etc/ssh/sshd_config`. I added these lines:
```
ListenAddress 192.168.1.1
Port 22
PermitRootLogin yes
Ciphers +chacha20-poly1305@openssh.com,aes256-gcm@openssh.com,aes256-ctr,aes256-cbc
```

On the ssh client, edit `~/.ssh/config` as needed. I needed to add:
`HostKeyAlgorithms       +rsa-sha2-512,rsa-sha2-256`

Trying to start a server and log in now will most certainly result in an error about "account expired". Fix that by editing `/etc/shadow` to remove the last "0" on the "admin" line:

Before:
`admin:$1$xxxxxxxxxxxxxxxxxxxxxxxxxxxx:0:0:99999:7:0:0:`

After:
`admin:$1$xxxxxxxxxxxxxxxxxxxxxxxxxxxx:0:0:99999:7:0::`

Or, instead of editing, run this:
`sed -i.bak.sshd '/^admin/ s!:0:$!::!' /etc/shadow`


#### Test to check if the server starts and works:
`/opt/sbin/sshd -d`

Use telnet to check that the right server is listening on the right port: `telnet 192.168.1.1 22` should show something like this:`SSH-2.0-OpenSSH_9.0`

ssh to the router from desktop/laptop client. Pay attention to the port number, and (optionally) use verbose mode (`ssh -v`) to double-check the server's version information. There should be a line that looks like this: `debug1: Remote protocol version 2.0, remote software version OpenSSH_9.0`

If needed, testing can be done via `/opt/sbin/sshd -d -o 'Port=22'`, or similar. Ultimately, all options should be set in `/opt/etc/ssh/sshd_config`.

If it doesn't work at this point, let us know what's broken and how you fixed it.

#### Configure the port numbers to what you really want

If it works at this point… Either leave the port numbers as-is, or…

Use the GUI to change the (dropbear) ssh to a different port, eg 2222. nb that port has to be available to make this change; kill any sshd process that may be listening on 2222, or whatever port number you're using here.

Then change `/opt/etc/ssh/sshd_config` to listen on port 22.

If listening ports are changed, expect to see "WARNING: REMOTE HOST IDENTIFICATION HAS CHANGED!" messages, when connecting for the first time, and edit the (client side!) `~/.ssh/known_hosts` accordingly.

Alternatively, on the client, edit `~/.ssh/config` to connect to port 2222, via a "Host" declaration.

Now, I've got sshd (openssh) running on port 22, and dropbear running on port 2222, just in case.

#### Next, changes need to survive reboots.

The "openssh-server" package installs an init.d start-up script, so we shouldn't have to worry about that.

Create a shell script `/jffs/bin/sshd-setup.sh` and make sure it's executable.
```
#!/bin/sh

## add "sshd" user
grep -q '^sshd:' /etc/passwd || {
    echo 'sshd:x:22:65534::/run/sshd:/dev/null' >> /etc/passwd
}

## make the "admin" account not expired
sed -i.bak.sshd '/^admin:/ s!:0:$!::!' /etc/shadow
```

Add this to `/jffs/scripts/init-start`, and make sure that's executable:
```
## set-up for sshd
/jffs/bin/sshd-setup.sh
```

#### Reboot, test, double-check everything.

#### Profit

More info on "scripts/init-start" here - https://github.com/RMerl/asuswrt-merlin.ng/wiki/User-scripts

If everything worked out, the router can be rebooted, openssh will be listening on your preferred port, and dropbear will be listening on a different port, just in case you need it.

I'm finding openssh much more reliable and stable than dropbear.