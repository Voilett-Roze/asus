I'd like to share a little project of mine. My TV can display a webpage in full screen accessible from the home screen via a launcher button. So I thought it would be convenient for friends an relatives who come by to be able to get the Guest Wifi QR code directly from my TV and join the network in a simple way.

Inspired by others threads and docs ([here](https://blog.tldnr.org/2019/11/11/Guest-Wifi/), [here](https://github.com/RMerl/asuswrt-merlin.ng/wiki/Setting-a-random-password-for-guest-wifi) and [there](https://linux.die.net/man/1/qrencode)), I made a little script that is launched every night by a cron task. Basically it does 2 things :

1. Rotate my guest wifi password every night with a new random value
1. Update a webpage hosted locally on lighttpd, that display the new password, in plain text and in a qr code

Questions and comments about this script on [this SNB forum thread](https://www.snbforums.com/threads/guest-wifi-random-password-qr-code-generator-for-display-on-smart-tv.75047/).

## Requirements

There are a few prerequisite for this to work:

### Entware

* First, it's necessary to have Entware on your router in order to install some other packages, and for that you need a USB storage device connected to it.

### Packages required :

* lighttpd, and optionnally lighttpd-mod-access to control who can access it (and other modules to your convenience but none are necessary, no php needed)
* qrencode : to convert WIFI password into QR code
* coreutils-base64 : to format qrcode into data uri

I'm not entering in the details of setting up entware and lighttpd, there are plenty of resources to help achieving that.

## Script

Here is the script. I put it on my usb key, as /mnt/MERLIN/rotateGuestPassword.sh, because I don't like to write too much on jffs. It produces a single basic html file and places it directly on the webserver.

```
#!/bin/bash

## CONFIGURATION OF THE SCRIPT

# List of guest  networks to use
# In this example I change password for the first 2.4GHz and the first 5Ghz network
# You can get the list of wireless networks with a command such as "nvram show | grep ssid | grep "^w""
# It will list the networks ssid, and the part you need is the one before the underscore
# You can set as many networks as you want/have on your router. But for a display on TV, a maximum of 2 will work the best with no need to scroll.
wl_list="wl0.1 wl1.1"

# Path to save the html_file on the webserver
html_file="/opt/share/www/index.html"

# Characters list for generated passwords. 
# Be careful with non alphanumeric characters that might break the password generation command... 
# Some people remove I,i and 1 to avoid errors, but if you are going to use the QR code, you don't mind...
char_list="ABCDEFGHIJKLMNPQRSTUVWXYZabcdefghijklmnpqrstuvwxyz123456789"

# Length of generated passwords
# Password should be between 8 and 63 characters
pw_length=20

## END OF CONFIGURATION OF THE SCRIPT

# Get a random password
getRandomPassword(){
    random_password=`cat /dev/urandom | env LC_CTYPE=C tr -dc "$char_list" | head -c $pw_length; echo;`    
}

# Start to write the temporary html file
cat <<EOF > "$html_file".tmp
<!DOCTYPE html>
<html>
    <head>
        <meta charset="UTF-8">
        <title>Home Guest WIFI</title>
        <style>
            html {
                background:#e9dcc3;             
                text-align:center;
                font-family:Verdana,Arial,Helvetica
            }
            .container {
                position: absolute;
                top: 0;
                bottom: 0;
                left: 0;
                right: 0;
                width: 100%;
                height: 660px;
                margin: auto;   
            }
            .item {
                margin: auto 50px;
                width: 660px;
                display: inline-block;
                height: 660px;
                background-color: #fff;
                border-top: solid 10px #e9dcc3;
                border-bottom: solid 10px #e9dcc3;
            }
            img {
                margin:auto;
                max-height:450px;
                max-width:450px;
            }
        </style>
    </head>
    <body>
        <div class="container">
EOF

# Loop on networks list
for wl in $wl_list
do
    #Generate a password   
    getRandomPassword
    
    # Get SSID of currently processed network   
   ssid=`nvram get "$wl"_ssid`
  
   # Set the new password of currently processed network   
   nvram set "$wl"_wpa_psk="$random_password"

   # Add block with for this network to html temp file
     cat <<EOF >> "$html_file".tmp
            <div class="item">
                <h1>Guest WIFI</h1>
EOF

# Add the qrcode as a Data uri image
    echo "                <img src=\"data:image/svg+xml;base64,"$(qrencode -o - "WIFI:S:$ssid;T:WPA;P:$random_password;;" -s 10 -t SVG | base64 | tr -d '\r\n')"\" />" >> "$html_file".tmp

# Finish to write the block
    cat <<EOF >> "$html_file".tmp
                <h2>
                    SSID : $ssid<br />
                    SECURITY : WPA / WPA2<br />
                    PASSWORD : $random_password
                </h2>
            </div>
EOF

done

# Finish to write the temporary html file
cat <<EOF >> "$html_file".tmp
        </div>
    </body>
</html>
EOF

# Commit the changes to survive a reboot   
nvram commit   

# Restart WIFI to apply changes
service restart_wireless

# Overwrite the final html file
mv -f "$html_file".tmp "$html_file"
```

### Cron task

A cron task run the script once a night. I set it up with the cru command

```
cru a rotateGuestPassword "0 4 * * * /mnt/MERLIN/rotateGuestPassword.sh"
```

## Accessing the webpage

With default lighttpd config, listenning on port 81, the generated webpage with the QR code is accessible through http://192.168.0.1:81/ assuming 192.168.0.1 is your router IP.

Then I just had to set up my tv to open this web page as a home screen.

And of course, unless you set up some access control, the webpage is accessible to any devices on the local network. You don't need to have a TV to display it.

## Disclaimer, notes and improvments

The script comes as is, I'm not offering any support or warranty. It works fine on my AX3000 with Merlin 386.3_2 but that's all I can say. It is a little dirty and minimal, but it's a single file, and it produces a single html file. For a tutorial purpose, it was simpler. There is plenty of room for improvments, feel free to comment or adapt.

The version of qrencode currently available on entware is not able to generate PNG files, hence the SVG format. It's of course possible to produce the svg file instead of a data uri and to save it on the websrv.

The html file could be build up from a template, with only the variables changed by the script. The html layout could be greatly improved.

Choosing a password length of more than 25 characters doesn't display well in the current html layout. Up to you to fix that if you are paranoid enough to need such a long password.