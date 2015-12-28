## About
With dnsmasq you are able to create custom domains within your network or route existing domains to different ip's. It's very handy when you want to create *home* web which will have web links to your NAS storage, printer and other clever things within your household. This can be also used to block access to domains by routing them to different ip address, so you can block advertising within some applications.    

## Adjust router configuration

1. Go to `Administration -> System`
2. Enable: `Enable JFFS custom scripts and configs` config option
3. Enable: `Enable SSH config option`

## Connect to your router

Connect to your router through SSH (you can use PUTTY on windows). Default IP address is `192.168.1.1`, use credentials as in web interface. ([How to use putty](https://www.google.sk/search?q=how%20to%20use%20putty))

## Edit dnsmasq config options

1. Create configuration file for dnsmasq: `touch /jffs/configs/dnsmasq.conf.add`
2. Edit configuration file: `vi /jffs/configs/dnsmasq.conf.add`. 

    *- for typing press `I`, to quit typing press `ESC`, to delete line press `ESC` and then write `dd` and press `ENTER`*
3. Add configuration for resolving domain names into `dnsmasq.conf.add`

    * Resolve one domain to IP, *Explanation: resolves `test.com` domain to ip `127.0.0.1` or `::1` when on ipv6*

            address=/test.com/127.0.0.1
            address=/test.com/::1

    * Resolve more domains to same IP, *Explanation: resolves listed domains to ip `127.0.0.1` (you can write more)*

            address=/test1.com/test2.com/127.0.0.1

    * **To save and quit editor** quit typing with `ESC` and write `:wq` and hit `ENTER`

## Last steps
1. Reboot rooter with `reboot` command in ssh or through web interface
2. Go to `Administration -> System` and disable `Enable SSH config option`

# More
For more dnsmasq options check [manual](http://www.thekelleys.org.uk/dnsmasq/docs/dnsmasq-man.html)
