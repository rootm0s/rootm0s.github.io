---
layout: post
title: FODHELPER UAC BYPASS
---

Fodhelper is a auto-elevated executable, which is signed by Microsoft. Fodhelper was introduced in Windows 10 (10240) to manage optional features, like keyboard settings.

This executable is vulnerable to class hijacking, it can be used to spawn our executable with High integrity level. We need to hijack the default value at "HKCU\Software\Classes\ms-settings\shell\open\command" with full path to our executable and create a new string value with the name DelegateExecute set to 0.

![_config.yml]({{ site.baseurl }}/images/fodhelper_procmon.JPG)

Once we created all the registry keys and values we can run fodhelper executable in order to trigger the UAC bypass. 

![_config.yml]({{ site.baseurl }}/images/fodhelper_cmd_pop.JPG)

This works from Windows 10 (10240) to **Unfixed**

### Full code at WinPwnage project:
[https://github.com/rootm0s/WinPwnage/blob/master/winpwnage/functions/uac/uac_fodhelper.py](https://github.com/rootm0s/WinPwnage/blob/master/winpwnage/functions/uac/uac_fodhelper.py)

### WinPwnage usage:
`python winpwnage.py --use uac --id 2 --payload c:\\windows\\system32\\cmd.exe`
