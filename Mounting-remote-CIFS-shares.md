You can mount remote SMB shares on your router.  The syntax will be something like this:

``mount \\\\192.168.1.100\\ShareName /cifs1 -t cifs -o "username=User,password=Pass"``

(backslashes must be doubled.)