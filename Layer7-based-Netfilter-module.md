Support for layer7 rules in iptables has been enabled on MIPS-based routers (RT-N66/AC66).  You will need to manually configure the iptables rules to make use of it - there is no web interface exposing this. The defined protocols can be found in /etc/l7-protocols.

To use it, you must first load the module:

    modprobe xt_layer7

An example iptable rules that would mark all SSH-related packets with the value "22", for processing later on in another chain:

    iptables -I FORWARD -m layer7 --l7proto ssh -j MARK --set-mark 22

These could be inserted in a firewall-start script, for example.

For more details on how to use layer7 filters, see the documentation on the project's website:

http://l7-filter.clearfoundation.com/