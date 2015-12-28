## Adjust router configuration

1. Go to `Administration` -> `System`
2. Enable: `Enable JFFS custom scripts and configs` config option
3. Enable: `Enable SSH config option`

## Connect to your router

Connect to your router through SSH (you can use PUTTY on windows). Default IP address is `192.168.1.1`, use credentials as in web interface. ([How to use putty](https://www.google.sk/search?q=how%20to%20use%20putty))

## Edit dnsmasq config options

1. Create configuration file for dnsmasq: `touch /jffs/configs/dnsmasq.conf.add`
2. Edit configuration file: `vi /jffs/configs/dnsmasq.conf. *- for typing press `I`, to quit typing press `ESC`, to delete line press `ESC` and then write `dd` and press `ENTER`*
3. Add configuration for resolving domain names into config 

    # Resolve one domain to IP, Explanation: resolves `test.com` domain to ip `127.0.0.1` or `::1` when on ipv6
    address=/test.com/127.0.0.1
    address=/test.com/::1

    # Resolve more domain to same IP, resolves test1.com and test2.com domains to ip `127.0.0.1` or `::1` when on ipv6
    address=/test1.com/test2.com/127.0.0.1
