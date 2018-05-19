Quite a few tricks can be done through the implementation of a few iptable rules either in the firewall-start scripts or the nat-start scripts.

These tricks will require first that you enable the JFFS partition, and then that you put your rules in one of the user scripts (either firewall-start or nat-start, depending on the case).

### Allow port forwarding to a service (like RDesktop) only from a specific IP

Let's say you want to create a port forward that will only accept connections from a specific IP (for example, you want to only allow RDesktop connection coming from IP 10.10.10.10).  First, make sure you do NOT forward that port on the Virtual server page.  Then, use a rule like this inside the nat-start script:

```sh
iptables --insert VSERVER 3 --table nat --proto tcp --match tcp --source 10.10.10.10 --dport 3389 --jump DNAT --to-destination 192.168.1.100
```
This will forward connections coming from 10.10.10.10 to your PC running RDesktop, on IP 192.168.1.100.

The same method can be used to allow forwarding SSH, FTP, etc...  Just adjust the --dport to match the service port you want to forward to.

### Configure a port forward with brute force detection

The following nat-start script will create a port forward that will only forward connections if there is no more than a set number of attempts within a period of time.  This example will forward SSH to an internal server sitting on 192.168.1.100, limiting connection attempts at 5 per period of 60 seconds:

```sh
#!/bin/sh

logger "firewall" "Applying nat-start rules"
# create a new chain SSHVSBFP
iptables --new SSHVSBFP --table nat
# add rule: add the source IP to the SSHVS match list table using the 'recent' match extension
iptables --append SSHVSBFP --table nat --match recent --set --name SSHVS --rsource
# add rule: deny if address has been seen in the SSHVS match list more than 4 times in the last 30 seconds
iptables --append SSHVSBFP --table nat --match recent --update --name SSHVS --seconds 60 --hitcount 5 --rsource --jump RETURN
# add rule: forward packets on port 22 to 192.168.1.100 using the DNAT target extension
iptables --append SSHVSBFP --table nat --proto tcp --dport 22 --match state --state NEW --jump DNAT --to-destination 192.168.1.100
# add the chain created above to the VSERVER chain and apply to interface eth0 (public interface)
iptables --insert VSERVER --table nat --in-interface eth0 --proto tcp --dport 22 --match state --state NEW --jump SSHVSBFP
```

Make sure to remove any existing port forward rule - this script will take care of creating the forward rule.

