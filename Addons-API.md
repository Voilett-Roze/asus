# Third party addons

## Introduction
Starting with 384.15, Asuswrt-Merlin now supports web integration for third party addons.  Up to a maximum of ten custom pages can be added on any existing section of the webui, as a new tab.

There is also a dedicated settings storage for addons, separate from nvram, and therefore not bound to its limitations (i.e. you can create new settings without having to recompile a firmware image).


## Locations
The _/jffs/addons/_ directory is the standard location.  You should create a directory there to store all your files.  Do not add files at the root of this directory.

The user-defined settings for all addons will be stored in _/jffs/addons/custom_settings.txt_.


## Custom pages
The firmware supports up to ten custom pages.  These pages should be mounted at boot time through a script, which should ideally be called from services-start.  You should put your custom page in the _/jffs/addons/my_addon/_ folder along with the install script.  Here is a sample of an install script, which will also insert your page as a new tab in the Tools section.

```
#!/bin/sh

source /usr/sbin/helper.sh

# Obtain the first available mount point in $am_webui_page
am_get_webui_page /jffs/addons/my_addon/MyPage.asp

if [ "$am_webui_page" = "none" ]
then
        logger "MyPage" "Unable to install Mypage"
        exit 5
fi
logger "MyPage" "Mounting MyPage as $am_webui_page"

# Copy custom page
cp /jffs/addons/my_addon/MyPage.asp /www/user/$am_webui_page

# Copy menuTree (if no other script has done it yet) so we can modify it
if [ ! -f /tmp/menuTree.js ]
then
        cp /www/require/modules/menuTree.js /tmp/
        mount -o bind /tmp/menuTree.js /www/require/modules/menuTree.js
fi

# Set correct return URL within your copied page
sed -i "s/MyPage.asp/$am_webui_page/g" /www/user/$am_webui_page

# Insert link at the end of the Tools menu.  Match partial string, since tabname can change between builds (if using an AS tag)
sed -i "/url: \"Tools_OtherSettings.asp\", tabName:/a {url: \"$am_webui_page\", tabName: \"My Page\"}," /tmp/menuTree.js

# sed and binding mounts don't work well together, so remount modified file
umount /www/require/modules/menuTree.js && mount -o bind /tmp/menuTree.js /www/require/modules/menuTree.js
```


## Custom settings
Since new nvram settings cannot be dynamically added without a firmware rebuild, and also because of size limitations on the newer Broadcom HND platform (where new variables are limited to values of 100 bytes max), a new settings repository was implemented for addons.  The _/jffs/addons/custom_settings.txt_ file will contain all these third party settings.  Each line will contain one setting, defined like this:

```
setting1 My First Setting
setting2 My Second Setting
```
Variable names should be limited to alphanumeric, the dash (-) and the underscore (_) character.  The maximum length for a name is 29 characters.

The variable content should in theory allow any 7-bit ASCII printable characters.    The size limit for the content is 2999 characters.  If you need more complex values (like strings with carriage returns), it's recommended to use base64 encoding, or to move the data to a separate file (if it's too large or too complex).  Note that you could write these files to the /www/user/ folder if read access from your web page is necessary.  In that case, make sure to either use a unique name (reusing your established namespace, for example), or to store data in a sub-directory.

The maximum total size of the whole repository is 8 KB.  Due to these size limitations, try to avoid storing large amounts of data there - use separate files instead if you need to.  If it gets any larger, settings will be truncated when processed by the web server.

You should define a namespace to ensure easy identification of your settings, since all addons will share the same repository.  This would look like this:

```
my_addon_version 3.0.2
my_addon_state enabled
diversion_version 2.0
diversion_whitelist /jffs/addons/diversion/my-whitelist.txt
```

### Shell usage
For shell script manipulation, helper.sh provide a pair of functions to retrieve and store content.

```
am_settings_set setting_name new_value
am_settings_get setting_name
````

Examples:

```
# set a value
am_settings_set addon_title Cool Addon 1.0
# retrieve it, in the $TITLE variable
TITLE=$(am_settings_get addon_tittle)
```

### Web usage
For web integration, you can access these settings within your page as a Javascript Object, by inserting the following near the start of your page:

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

### MyService start
/jffs/addons/my_addon/myservice.sh $*
### MyService end
```

> Note: Avoid adding complex scripts to the system _/jffs/scripts/*_ scripts.  Instead, put your script in your _/jffs/addons/my_addon/_ folder, and call that script from the relevant system script.  That way, you can easily develop an uninstaller that would look for those start/end lines, and remove the entire block without affecting other installed addons.


## Example custom page
Here is a simple example page, which you can use as a starting point.

```
<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Transitional//EN" "http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd">
<html xmlns="http://www.w3.org/1999/xhtml">
<head>
<meta http-equiv="X-UA-Compatible" content="IE=Edge">
<meta http-equiv="Content-Type" content="text/html; charset=utf-8" />
<meta HTTP-EQUIV="Pragma" CONTENT="no-cache">
<meta HTTP-EQUIV="Expires" CONTENT="-1">
<link rel="shortcut icon" href="images/favicon.png">
<link rel="icon" href="images/favicon.png">
<title>Test page</title>
<link rel="stylesheet" type="text/css" href="index_style.css">
<link rel="stylesheet" type="text/css" href="form_style.css">
<script language="JavaScript" type="text/javascript" src="/state.js"></script>
<script language="JavaScript" type="text/javascript" src="/general.js"></script>
<script language="JavaScript" type="text/javascript" src="/popup.js"></script>
<script language="JavaScript" type="text/javascript" src="/help.js"></script>
<script type="text/javascript" language="JavaScript" src="/validator.js"></script>
<script>


var custom_settings = <% get_custom_settings(); %>;

function initial(){
        show_menu();

        if (custom_settings.diversion_path == undefined)
                document.getElementById('diversion_path').value = "/tmp/default";
        else
                document.getElementById('diversion_path').value = custom_settings.diversion_path;
}

function applySettings(){
        /* Retrieve value from input fields, and store in object */
        custom_settings.diversion_path = document.getElementById('diversion_path').value;

        /* Store object as a string in the amng_custom hidden input field */
        document.getElementById('amng_custom').value = JSON.stringify(custom_settings);

        /* Apply */
        showLoading();
        document.form.submit();
}
</script>

                            
</head>
<body onload="initial();"  class="bg">
<div id="TopBanner"></div>
<div id="Loading" class="popup_bg"></div>
<iframe name="hidden_frame" id="hidden_frame" src="" width="0" height="0" frameborder="0"></iframe>
<form method="post" name="form" action="start_apply.htm" target="hidden_frame">
<input type="hidden" name="current_page" value="MyPage.asp">
<input type="hidden" name="next_page" value="MyPage.asp">
<input type="hidden" name="group_id" value="">
<input type="hidden" name="modified" value="0">
<input type="hidden" name="action_mode" value="apply">
<input type="hidden" name="action_wait" value="5">
<input type="hidden" name="first_time" value="">
<input type="hidden" name="action_script" value="">
<input type="hidden" name="preferred_lang" id="preferred_lang" value="<% nvram_get("preferred_lang"); %>">
<input type="hidden" name="firmver" value="<% nvram_get("firmver"); %>">
<input type="hidden" name="amng_custom" id="amng_custom" value="">

<table class="content" align="center" cellpadding="0" cellspacing="0">
<tr>
<td width="17">&nbsp;</td>
<td valign="top" width="202">
<div id="mainMenu"></div>
<div id="subMenu"></div>
</td>
<td valign="top">
<div id="tabMenu" class="submenuBlock"></div>
<table width="98%" border="0" align="left" cellpadding="0" cellspacing="0">
<tr>
<td align="left" valign="top">
<table width="760px" border="0" cellpadding="5" cellspacing="0" bordercolor="#6b8fa3" class="FormTitle" id="FormTitle">
<tr>
<td bgcolor="#4D595D" colspan="3" valign="top">
<div>&nbsp;</div>
<div class="formfonttitle">Test page - Custom settings</div>
<div style="margin:10px 0 10px 5px;" class="splitLine"></div>
<div class="formfontdesc"><#1838#></div>

<table width="100%" border="1" align="center" cellpadding="4" cellspacing="0" bordercolor="#6b8fa3" class="FormTable">

        <tr>
                <th>Diversion path</th>
                <td>
                        <input type="text" maxlength="100" class="input_25_table" id="diversion_path" autocorrect="off" autocapitalize="off">
                </td>
        </tr>
</table>
<div class="apply_gen">
        <input name="button" type="button" class="button_gen" onclick="applySettings();" value="Apply"/>
</div>
</form>

<div>
<table class="apply_gen">
<tr class="apply_gen" valign="top">
</tr>
</table>
</div>
</td>
</tr>
</table>
</td>
</tr>
</table>
</td>
<td width="10" align="center" valign="top"></td>
</tr>
</table>
<div id="footer"></div>
</body>
</html>
```
