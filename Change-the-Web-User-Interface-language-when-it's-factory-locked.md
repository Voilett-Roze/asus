In some countries (such as Japan), some ASUS routers are set to allow only one language in the webui, and you can't change it. 

However it's possible to use the below script to change the language on demand via SSH. To set the language at reboot/startup, the script must be added to /jffs/scripts/init-start.

First you need to enable custom scripts by ticking "Enable JFFS custom scripts and configs" in Administration > System

Then, you have to create the following script as /jffs/scripts/set-webui-lang.sh

```
#!/bin/sh

############################################################################
#
## Usage: set-webui-lang.sh [options] ARG1
## Changes the language of the router user interface 
## Script designed for asuswrt-merlin 
## Tested only on Asus RT-AX3000 with asuswrt-merlin v384.17
## Author : joomlafab
## 
## Options:
##   -p            Provide a prompt to choose language
##                 ARG1 ignored if used
##
##   -q            Quiet mode, no messages echoed
##
##   -f            Reset the UI to original factory language
##                 ARG1 and -p ignored if used
##
## Arguments
##   ARG1          Language code (2 letters, case insensitive)
##                 If not provided, or provided ARG1 doesn't match
##                 an existing dictionnary, will apply DEFAULT_DICT
#
############################################################################

# Default dictionnary to use 
DEFAULT_DICT="EN"

# Get the original factory language
FACTORY_DICT=$(nvram get preferred_lang)

# List all available dictionnaries
AVAILABLE_LANGUAGES=$(ls /www/ | grep dict | cut -d'.' -f1 | xargs | tr '\s' ', ')

# Get options
for param in "$@"; do 
   case $param in
      -p)
         prompt=true
         ;;
      -q)
	     quiet=true
		 ;;
      -f)
         factory=true
         ;;
      *)
		 if [[ -z "$inputDict" ]]; then
		    inputDict=$(echo $param | tr 'a-z' 'A-Z' | grep -o -w -E '^[A-Z]{2}')
         fi
		 ;;
	esac	 
done

############################################################################
#
#   log             - echo a log message if quiet mode wasn't set
#
############################################################################
log()
{
   if [[ -z "$quiet" ]] ; then
      echo "$1"
   fi
}

# Choose the proper dictionnary according to options
if [[ "$factory" = true ]]; then
   inputDict="$FACTORY_DICT"
elif [[ "$prompt" = true ]]; then
   echo "Available AVAILABLE_LANGUAGES : $AVAILABLE_LANGUAGES"
   while true; do
      read -p "Input the language code, then press Enter : " inputDict
      inputDict=$(echo $inputDict | tr 'a-z' 'A-Z' | grep -o -w -E '^[A-Z]{2}')
      dictFile="/www/$inputDict.dict"
      if [[ -n $inputDict ]] && [[ -f $dictFile ]]; then
         break;
      fi
      echo -e "Wrong language code.\nAvailable AVAILABLE_LANGUAGES : $AVAILABLE_LANGUAGES";
   done
elif [[ -z "$inputDict" ]]; then
   inputDict="$DEFAULT_DICT"
   log "No valid language code supplied."
   log "Applying default $DEFAULT_DICT dictionnary."
fi

# Verify that selected dictionnary exists
dictFile="/www/$inputDict.dict"
while [[ ! -f $dictFile ]]; do
   if [[ "$inputDict" = "$DEFAULT_DICT" ]]; then
      log "Cannot find default dictionnary $DEFAULT_DICT. No changes applied."
	  exit;
   fi
   log "Cannot find $inputDict dictionnary.";
   log "Applying default $DEFAULT_DICT dictionnary."
   inputDict="$DEFAULT_DICT"
   dictFile="/www/$DEFAULT_DICT.dict"
done
log "Change user interface language to $inputDict."

# Unmount existing mounted dictionnaries
# Done in a loop because it seems possible to mount them several times
for mountPoint in $(mount | cut -d' ' -f3 | grep "/www/$FACTORY_DICT.dict"); do
	umount "$mountPoint"
done

# Mount selected language dictionnaries
if [[ "$inputDict" != "$FACTORY_DICT" ]]; then
   mount -o bind $dictFile "/www/$FACTORY_DICT.dict"
fi

# Restart UI
service restart_httpd > /dev/null

log "Done."
```

Now you can call the script to change the language, with various options

```
# Call the script with a prompt to choose the language
/jffs/scripts/set-webui-lang.sh -p
```

```
# Call the script to set the webui to Language XX
# If wrong code, DEFAULT_DICT will be applied
# DEFAULT_DICT is set at top of the script
/jffs/scripts/set-webui-lang.sh XX
```

```
# Call the script to reset the webui to factory language
/jffs/scripts/set-webui-lang.sh -f
```

**Attention**, default factory language will be back at reboot. In order to avoid that, you can add a call to this script to /jffs/scripts/init-start. Change "EN" to whatever available language you want to set.

```
/jffs/scripts/set-webui-lang.sh -q EN
```

This one liner will do that for you, (and create /jffs/scripts/init-start if necessary). Change "EN" to whatever available language you want to set.

`[[ ! -f /jffs/scripts/init-start ]] && echo -e "#!/bin/sh\n" > /jffs/scripts/init-start && chmod +x /jffs/scripts/init-start ; if ! grep -q "set-webui-lang.sh" /jffs/scripts/init-start; then echo -e "/jffs/scripts/set-webui-lang.sh -q EN\n" >> /jffs/scripts/init-start; fi`