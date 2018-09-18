The router can be accessed over SSH (through Dropbear).  Password-based login will use the same username and password as telnet/web access. You can also optionally insert a RSA or ECDSA public key there for keypair-based authentication.  Finally, there is also an option to make SSH access available over WAN.

## Connecting to the router via SSH
**Requirements**
* ssh client (examples: macOS via Terminal, Linux via Terminal)

Make a connection to the router with this command:  
  
`# ssh <user>@<router>`  
> Change `<user>` with the username used for web access to the router.  
> Change `<router>` to the IP of the router in it's LAN side, for local access or WAN side, if connecting remotely.  
  
If the connection from the client to the router is new, you will get a message like this:  

`The authenticity of host 'ROUTER IP' cannot be established.
DSA key fingerprint is 02:89:30:31:b0:f3:2d:9b:01:3f:b3:a7:38:e7:b2:25.
Are you sure you want to continue connecting (yes/no)?`  
  
Just input `yes` to continue with the connection.  

The terminal will ask for the password afterwards. Just put the web access password for the router.  

The terminal will establish the connection via SSH and you can start to input commands.