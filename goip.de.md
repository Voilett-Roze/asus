#!/bin/sh
curl -k "https://www.goip.de/setip?
    username=Youname& \
    password=Yourpassword& \
    subdomain=Yoursubdomain.goip.de& \
    ip=${1}"
if [ $? -eq 0 ]; then 
	/sbin/ddns_custom_updated 1
else
	/sbin/ddns_custom_updated 0
fi