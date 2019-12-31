# Third party addons

Starting with 384.15, Asuswrt-Merlin now supports web integration for third party addons.  Up to a maximum of five custom pages can be added at any location on the webui.

There is also a dedicated settings storage for addons, separate from nvram, and therefore not bound to its limitations (i.e. you can create new settings without having to recompile a firmware image).


## Locations
The _/jffs/addons/_ directory is the standard location.  You should create a directory there to store all your files.

The user-defined settings for all addons will be stored in _/jffs/addons/custom_settings.txt_.


## Custom pages
The firmware supports up to five custom pages.  These pages should be mounted at boot time through a script, which should ideally be called from services-start.  You should put your custom page in the _/jffs/addons/my_addon/_ folder along with the install script.  Here is a sample of an install script, which will also insert your page as a new tab in the Tools section.

```
#!/bin/sh

source /usr/sbin/helper.sh

# Locate the first available mount point
MyPage=$(get_webui_page)
if [ $MyPage = "none" ]
then
        logger "MyPage" "Unable to install Mypage"
        exit 5
fi
logger "MyPage" "Mounting MyPage as $MyPage"

# Copy custom page
cp /jffs/addons/my_addon/MyPage.asp /www/user/$MyPage

# Copy menuTree (if no other script has done it yet) so we can modify it
if [ ! -f /tmp/menuTree.js ]
then
        cp /www/require/modules/menuTree.js /tmp/
        mount -o bind /tmp/menuTree.js /www/require/modules/menuTree.js
fi

# Set correct return URL within your copied page
sed -i "s/MyPage.asp/$MyPage/g" /www/user/$MyPage

# Insert link at the end of the Tools menu.  Match partial string, since tabname can change between builds (if using an AS tag)
sed -i "/url: \"Tools_OtherSettings.asp\", tabName:/!b;n;c{url: \"$MyPage\", tabName: \"My Page\"}," /tmp/menuTree.js

# sed and binding mounts don't work well together, so remount modified file
umount /www/require/modules/menuTree.js && mount -o bind /tmp/menuTree.js /www/require/modules/menuTree.js
```


## Custom settings
Since new nvram settings cannot be dynamically added without a firmware rebuild, and also because of size limitations on the newer Broadcom HND platform, a new repository was implemented.  The _/jffs/addons/custom_settings.txt_ file will contain third party settings.  The max total size is about 8 KB.  Each line will contain one setting, defined like this:

```
setting1 My First Setting
setting2 My Second Setting
```

You should define a namespace to ensure easy identification of your settings, since all addons will share the same repository.  This would look like this:

```
my_addon_version 3.0.2
my_addon_state enabled
diversion_version 2.0
diversion_whitelist /jffs/addons/diversion/my-whitelist.txt
```


Within your page, you can access these settings as a Javascript Object, by inserting the following near the start of your page:

```
var custom_settings = <% get_custom_settings(); %>;
```

In the HTML section, define your input fields, as you would do for fields using nvram settings:

```
<tr>
   <th>Diversion path</th>
   <td>
      <input type="text" maxlength="100" class="input_25_table" id="diversion_whitelist" autocorrect="off" autocapitalize="off">
   </td>
</tr>
```

Also define an hidden amng_custom input field, which will be used for sending the settings back when applying them:

```
<input type="hidden" name="amng_custom" id="amng_custom" value="">
```

Then, in the initial() function, retrieve the values from the object, and assign them to your fields:

```
    if (custom_settings.diversion_whitelist == undefined)
        document.getElementById('diversion_whitelist').value = "/jffs/addons/diversion/diversion-white.txt";
    else
       document.getElementById('diversion_whitelist').value = custom_settings.diversion_whitelist;
```


In the apply() function called when the user clicks on the Apply button, retrieve the field values, store them back in the object, convert the object into a string and assign it to the hidden amng_custom field, then submit the form:

```
function applySettings(){
        /* Retrieve value from input fields, and store in object */
        custom_settings.diversion_whitelist = document.getElementById('diversion_whitelist').value;

        /* Store object as a string in the amng_custom hidden input field */
        document.getElementById('amng_custom').value = JSON.stringify(custom_settings);

        /* Apply */
        showLoading();
        document.form.submit();
}
```

## Custom service
You will probably need to run some script on the router to act on those settings change.  The service-event script is the place to do so, as it will be run even for events unrecognized by the router, allowing you to design your own service
handlers.

In your page, configure your service restart.  For example, let's call it "myservice":

```
<input type="hidden" name="action_script" value="restart_myservice">
```

Create a _/jffs/addons/my_addon/myservice.sh_ script, containing something like this:

```
#!/bin/sh

TYPE=$1
EVENT=$2

if [ "$EVENT" = "myservice" -a "$TYPE" = "restart" ]
then
        logger -t "myservice" "Restarting my service..."
        # do your stuff here, access /jffs/addons/custom_settings.txt, etc...
fi
```

And in _/jffs/scripts/service-event_, call your script:

```
#!/bin/sh

# MyService
/jffs/addons/my_addon/myservice.sh $*
```


## Notes
- When designing your addon, decide on a namespace to use as the prefix for all
  your settings: my_addon_version, my_addon_state, etc...
- Do not put files at the root of _/jffs/addons/_.  Instead, create a folder there
  to store all your files.
- Avoid adding complex scripts to the system _/jffs/scripts/*_ scripts.  Instead, put
  your script in your _/jffs/addons/my_addon/_ folder, and call that script from
  the relevant system script, like this:
```
    ### my_addon start
    /jffs/addons/my_addon/launch.sh
    ### my_addon end
```
  That way, your can easily develop an uninstaller that would look for those start/end lines, and remove the entire block without affecting other installed addons.
- Avoid storing unnecessarily large amounts of data in the custom_settings repository, as its size is limited.
- The size limit for a setting's name is 29 character.  The size limit for the assigned value is 2999 characters.  The maximum total size of the whole repository is 8 KB.
