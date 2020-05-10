## DRAFT- In Progress

### About
QoS is a wonderful feature to let you optimize your network usage to prioritize the most important traffic on your network.  Having problems with Netflix stuttering when you're downloading updates?  Gaming pings suffering when your roommate is watching YouTube?  You need QoS.

Different types of QoS


### Configuration

For details on queueing discipline see TK link to queuing page

Link to FreshJRQoS

### Results


### Availability

Need help here to fill in models where Adaptive QoS is available

### Known Issues

#### Adaptive QoS breaks DHCP renewal

If you are having problems with your WAN connection dropping or DHCP failing to renew, you may be suffering from this problem.  Asus Adaptive QoS puts in place an iptables rule that blocks DHCP renewal when your lease expires.  If you are on a generous ISP which sets leases for days or weeks, you would not notice.  If you are on a less generous ISP like AT&T which sets DHCP lease for 10 minutes, your WAN connection will drop every 10 minutes.

##### Diagnosis

##### Changing Iptables

##### Testing

