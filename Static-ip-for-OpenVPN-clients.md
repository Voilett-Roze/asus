Sometimes, it can be useful to have VPN clients with static ip, for instance when the client is a backup destination for rsync. Here's how to do it on Asus router with Merlin firmware.
  
# 1. Setup the router
## 1.1. Generate certs with easy-rsa
You need to generate cert for the server and unique certs for each client using easy-rsa tool. To do so, just follow this tutorial:  
https://github.com/RMerl/asuswrt-merlin.ng/wiki/Generating-OpenVPN-keys-using-Easy-RSA  
Once it is done, just get all the files generated in the [folder you've chosen]/easy-rsa/keys. You can use WinSCP for that.

## 1.2. Use the generated certs in the router
In the keys folder you've downloaded, there's 3 files for each clients (.csr, .crt, .key), 3 for the server (.csr, .crt, .key), and 3 files related to the certificate authority (ca.crt, ca.key, dh1024.pem). You can ignore the other files.
  
You now have to use the certs in the server. To do so, in the GUI of the router, go to VPN > VPN Server > Select your server (1 or 2), go to advanced settings, 
![VPN settings 1](https://i.imgur.com/K2lebXl.png)  
  
Then edit the "Keys and Certificates".  
![VPN settings 2](https://i.imgur.com/Svy1cqG.png)  
- In the Certificate Authority field, paste the content of the ca.crt file.  
- In the Server Certificate, paste the content of the server.crt file (only from -----BEGIN CERTIFICATE----- to -----END CERTIFICATE-----, including those two lines).  
- In the Server Key, paste the content of the server.key file.  
- Finally, in the Diffie Hellman parameters, paste the content of the dh1024.pem.  
  
## 1.3. Setup ifconfig-pool-persist
Still in the advanced settings in the GUI, add this line in the custom configuration :
```shell
ifconfig-pool-persist /jffs/configs/openvpn/ipp.txt
```  
Finally, still on this screen, select yes for the "Manage Client-Specific Options", we'll need this for a later step.  
  
In a terminal, we'll create the ipp.txt file. So:
```shell
vi /jffs/configs/openvpn/ipp.txt
```
Then type i to type text and you'll have to create a file like this one:
```shell
client1,10.8.0.x
client2,10.8.0.x
client3,10.8.0.x
```
wth the static ip adresses you want for each client. Use the common names that you've set using easy-rsa. Press ESC then type ZZ to exit vi.

# 1.4. Create usernames and passwords for each clients
Go back to general settings in the VPN settings in the router GUI. Create usernames and passwords for each clients.

# 1.5. Create custom client config files
We now have to create ccd files for each client. To do so, create a file per client named after the common name set with easy-rsa in /jffs/configs/openvpn/ccd1.  
In this file, just type:
```shell
ifconfig-push 10.8.0.X 255.255.255.0
```
with the static ip adress you want for this client, obviously the same adress than in the ipp.txt file. Use vi to create this file.  
  
# 2. Configure the clients
The server is now set. Back in the router GUI, in the VPN Server page, click on Export OpenVPN Configuration file. Save the client1.ovpn file and edit it. 
  
![ovpn file](https://i.imgur.com/hl4HeN7.png)
  
In the end of the file, paste the content of the .crt file of your first client between the <cert></cert> tags, and the content of the .key file of your first client between the <key></key> tags.  
Save the file and then use it to connect to your VPN.  
  
Enjoy. 