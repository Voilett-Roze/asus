This guide provides an easy configuration to access the web-based UI of your
xDSL/Cable modem connected to the WAN port of your Asus router. **Important!** -
This only works if you are able to access the modem's configuration (some ISPs
have restrictions on their equipment). For this to work and to avoid issues,
your modem and router will be configured to utilize different subnets.

1. Change your modem's LAN IP address to `192.168.0.1`. Here's an example from a
   TP-Link ADSL modem:
   
   ![Modem LAN IP](https://i.postimg.cc/g0YZWPpf/Captura-47.png)
   
2. Go to _LAN -> LAN IP_. Ensure your router's LAN is on a different subnet; by
   default it will be (the default IP address is `192.168.1.1`). Click _Apply_.
   
   ![Router LAN IP](https://i.postimg.cc/zDBZRkv3/Captura-48.png)
   
3. Test if you can access the modem's interface. Open your browser and enter
   `http://192.168.0.1`
   
   ![Browser](https://i.postimg.cc/rFcRJHCX/Captura-50.png)

4. If the above step fails, go to _WAN -> Internet Connection -> WAN IP Setting_
   (section). Assign your router an IP from the modem's subnet by applying the 
   following settings:

   | WAN IP Setting               |                 |
   | ---------------------------- | --------------- |   
   | Get the WAN IP automatically | ⚪️ Yes 🔘 No    |
   | IP Address                   | `192.168.0.2`   |
   | Subnet Mask                  | `255.255.255.0` |
   | Default Gateway              | `192.168.0.1`   |

   and click _Apply_ at the bottom.
   
   ![Router WAN IP](https://i.postimg.cc/XJrLkTG4/Captura-49.png)
   
   ![Apply Button](https://i.postimg.cc/90wM7mLJ/Captura-48.png)
   
   Then retry step (3) above.

**That's it !**

If the above does not work, you can assign a virtual interface to eth0 - the WAN port. Keep your WAN config to "Automatic IP" then SSH into your merlin router, and use this command:

``` ifconfig eth0:1 192.168.0.2 netmask 225.225.225.0 ```

What this does is you assign a virtual interface to modem subnet 192.168.0.1 used in the example above. For the majority of us, it might be 192.168.1.1
Do check before you assign the interface. After assigning, go to 192.168.0.1 (your modem webui page) to test.