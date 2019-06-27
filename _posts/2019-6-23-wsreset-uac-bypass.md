---
layout: post
title: WSRESET UAC BYPASS
---

WSReset is used to reset Windows Store settings. This executable is signed by Microsoft and has "autoElevate" property set to true in the manifest.

This executable is vulnerable to class hijacking, it can be used to spawn our executable with High IL. We need to hijack the default value at "HKCU\Software\Classes\AppX82a6gwre4fdg3bt635tn5ctqjf8msdd2\Shell\open\command" with full path to our executable and create a new string value with the name DelegateExecute set to 0.

![_config.yml]({{ site.baseurl }}/images/wsreset_procmon.JPG)

Once we created all the registry keys and values we can run WSReset executable in order to trigger the UAC bypass. WSReset has to finish loading, can take a few seconds before the high IL process is created.

Tested on Windows 10 (17134)

### Full code at WinPwnage project:
[https://github.com/rootm0s/WinPwnage/blob/master/winpwnage/functions/uac/uac_wsreset.py](https://github.com/rootm0s/WinPwnage/blob/master/winpwnage/functions/uac/uac_wsreset.py)

### WinPwnage usage:
`python winpwnage.py --use uac --id 20 --payload c:\\windows\\system32\\cmd.exe`
