Asuswrt-Merlin includes a cutdown version of sendmail which can be used to send mail from the router.  Basic usage will look like this:

```
echo "This is a test email." | /usr/sbin/sendmail -S"smtp.yourisp.com" -f"router@domain.com" you@domain.com
```

You can send a more proper Email by writing the mail to a file, and then piping it to sendmail.  The following example, if saved as a wan-start script, will make your router send you an Email whenever the WAN interface comes up, containing the WAN IP:

```
#!/bin/sh
SMTP="smtp.yourisp.com"
FROM="router@domain.com"
FROMNAME="Your Router"
TO="you@domain.com"

echo "Subject: WAN state notification" >/tmp/mail.txt
echo "From: \"$FROMNAME\"<$FROM>" >>/tmp/mail.txt
echo "Date: $(date -R)" >>/tmp/mail.txt
echo "I just got connected to the Interwebz." >>/tmp/mail.txt
echo "My new IP is: $(nvram get wan0_ipaddr)" >>/tmp/mail.txt
echo "" >>/tmp/mail.txt
echo "--- " >>/tmp/mail.txt
echo "Your friendly router." >>/tmp/mail.txt
echo "" ­­>>/tmp/mail.txt

cat /tmp/mail.txt | /usr/sbin/sendmail -S"$SMTP" -f"$FROM" $TO

rm /tmp/mail.txt
```

If your SMTP server requires authentication, you can pass them as additional arguments.  To see the available options, just run "sendmail -h".

***
If you don't have a smtp email account from your ISP, [MailJet's SMTP relay serice](https://www.mailjet.com/feature/smtp-relay/) could be the solution, it offers 200 mails for free a day. MailJet is a European based company in France, Paris.

Now just fill your _wan-start script_ with the following lines but don't forget to replace the according lines with the data of your SMTP relay server (e.g. MailJet) and the email address where to receive notifications.
```
#!/bin/sh
SMTP="in-v3.mailjet.com:587"
FROM="your-mailjet-fromaddress"
FROMNAME="ASUS ROUTER"
TO="your-email-address"

ntpclient -h pool.ntp.org -s &> /dev/null
sleep 5

echo "Subject: WAN state notification" >/tmp/mail.txt
echo "From: \"$FROMNAME\"<$FROM>" >>/tmp/mail.txt
echo "Date: $(date -R)" >>/tmp/mail.txt
echo "" >>/tmp/mail.txt
echo "I just got connected to the internet." >>/tmp/mail.txt
echo "" >>/tmp/mail.txt
echo "My WAN IP is: $(nvram get wan0_ipaddr)" >>/tmp/mail.txt
echo "Uptime is: $(uptime | cut -d ',' -f1 | sed 's/^.\{12\}//g')" >>/tmp/mail.txt
echo "" >>/tmp/mail.txt
echo "---- " >>/tmp/mail.txt
echo "Your friendly router." >>/tmp/mail.txt
echo "" >>/tmp/mail.txt

cat /tmp/mail.txt | /usr/sbin/sendmail -S"$SMTP" -f"$FROM" $TO -au"your-MailJet-username" -ap"your-MailJet-password"

rm /tmp/mail.txt
```

***

It's possible to send emails even from **Gmail** account through openssl (thanks [Nerre](http://forums.smallnetbuilder.com/member.php?u=15302)), first we need to download a trusted certificate
```
wget -c -O /jffs/configs/google_root.pem https://pki.goog/roots.pem --no-check-certificate
```
Now fill your _wan-start script_ with the following text but don't forget to replace **FROM**, **AUTH**, **PASS** and **TO** values only in first six lines of the script
```
#!/bin/sh
FROM="your-gmail-address"
AUTH="your-gmail-username"
PASS="your-gmail-password"
FROMNAME="Your Router"
TO="your-email-address"

ntpclient -h pool.ntp.org -s &> /dev/null
sleep 5

echo "Subject: WAN state notification" >/tmp/mail.txt
echo "From: \"$FROMNAME\"<$FROM>" >>/tmp/mail.txt
echo "Date: $(date -R)" >>/tmp/mail.txt
echo "" >>/tmp/mail.txt
echo "I just got connected to the internet." >>/tmp/mail.txt
echo "" >>/tmp/mail.txt
echo "My WAN IP is: $(nvram get wan0_ipaddr)" >>/tmp/mail.txt
echo "Uptime is: $(uptime | cut -d ',' -f1 | sed 's/^.\{12\}//g')" >>/tmp/mail.txt
echo "" >>/tmp/mail.txt
echo "---- " >>/tmp/mail.txt
echo "Your friendly router." >>/tmp/mail.txt
echo "" >>/tmp/mail.txt

cat /tmp/mail.txt | sendmail -H"exec openssl s_client -quiet \
-CAfile /jffs/configs/google_root.pem \
-connect smtp.gmail.com:587 -tls1 -starttls smtp" \
-f"$FROM" \
-au"$AUTH" -ap"$PASS" $TO 

rm /tmp/mail.txt
```
After saving the script disconnect and connect again to internet or reboot router and check your email inbox.
![mail](http://i47.tinypic.com/10drrs6.png)
Please post issues here http://forums.smallnetbuilder.com/showthread.php?t=8190