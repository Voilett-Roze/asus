Here is an update notification with Pushbullet/Pushover/Mail Notifications feel free to modify and append upon this script, set the script to location below and make it executable and in the webui in the meny "Tools > Other Settings" set the value "New firmware version check" to "Yes" the script will check every 48 hours.
 
> /jffs/scripts/update-notification

> chmod +x /jffs/scripts/update-notification 


```
#!/bin/sh
use_email="disabled"                                    # enabled / disabled (default: disabled)
use_pushbullet="disabled"                               # enabled / disabled (default: disabled)
use_pushover="disabled"                                 # enabled / disabled (default: disabled)

#Pushbullet/Pushover settings
pushbullet_token=""                                     # Your access token here (https://docs.pushbullet.com/)
pushover_token=""                                       # Your access token here (https://pushover.net/api)
pushover_username=""                                    # Pushover User ID (the user/group key (not e-mail address often referred to as USER_KEY)

# email settings
SMTP="smtp.gmail.com"
PORT="465"
USERNAME=""
PASSWORD=""

# Mail Enveloppe
FROM_NAME=""
FROM_ADDRESS=""
TO_NAME=""
TO_ADDRESS=""

### Do not change below
# Retrieve version
TMPVERS=$(nvram get webs_state_info)
VERS=${TMPVERS:5:3}.${TMPVERS:8:10}
ROUTER_IP=$(nvram get lan_ipaddr)

email_message () {
echo "From: \"$FROM_NAME\" <$FROM_ADDRESS>" > /tmp/mail.txt
echo "To: \"$TO_NAME\" <$TO_ADDRESS>" >> /tmp/mail.txt
echo "Subject: New router firmware notification" >> /tmp/mail.txt
echo "" >> /tmp/mail.txt
echo "New firmware version $VERS is now available for your router at $ROUTER_IP." >> /tmp/mail.txt
curl --url smtps://$SMTP:$PORT \
  --mail-from "$FROM_ADDRESS" --mail-rcpt "$TO_ADDRESS" \
  --upload-file /tmp/mail.txt \
  --ssl-reqd \
  --user "$USERNAME:$PASSWORD" --insecure
rm /tmp/mail.txt
}

pushover_message () {
curl -s \
  --form-string "token=$pushover_token" \
  --form-string "user=$pushover_username" \
  --form-string "message=New firmware version $VERS is now available for your router at $ROUTER_IP." \
  https://api.pushover.net/1/messages.json
}

pushbullet_message () {
text="New firmware version $VERS is now available for your router at $ROUTER_IP."
title="$USER@$HOSTNAME"
curl -s -u $pushbullet_token: -X POST https://api.pushbullet.com/v2/pushes --header 'Content-Type: application/json' --data-binary '{"type": "note", "title": "'"$title"'", "body": "'"$text"'"}' >/dev/null 2>&1
}

     if [ $use_pushbullet = "enabled" ]; then
        pushbullet_message
     fi
     if [ $use_pushover = "enabled" ]; then
        pushover_message
     fi
     if [ $use_email = "enabled" ]; then
        email_message
     fi
```