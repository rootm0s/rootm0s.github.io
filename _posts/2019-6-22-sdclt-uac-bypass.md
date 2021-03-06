---
layout: post
title: SDCLT UAC BYPASS
---

Sdclt is a Microsoft executable file which was introduced in Windows 7 (7600) to allow users to perform backups or restore a backup. This executable is signed by Microsoft and has "autoElevate" property set to true in the manifest. You can verify the manifest using [Sigcheck](https://docs.microsoft.com/en-us/sysinternals/downloads/sigcheck) from Sysinternals.

This **does not** work on Windows 7 due to manifest has requestedExecutionLevel set to "AsInvoker", preventing auto-elevation from a Medium IL process. 

When sdclt executable is called from a Medium IL process, it runs another process of sdclt with high integrity level. This process calls control.exe which is located inside system32 directory.

Control executable is vulnerable to class hijacking, it can be used to spawn our executable with High IL. We need to hijack the default value at "HKCU\Software\Classes\Folder\shell\open\command" with full path to our executable and create a new string value with the name DelegateExecute set to 0.

![_config.yml]({{ site.baseurl }}/images/sdclt_control_procmon.JPG)

Once we created all the registry keys and values we can run sdclt executable in order to trigger the UAC bypass.

![_config.yml]({{ site.baseurl }}/images/sdclt_cmd_pop.JPG)

This works from Windows 10 (14393) to **Unfixed**

### Full code at WinPwnage project:
[https://github.com/rootm0s/WinPwnage/blob/master/winpwnage/functions/uac/uacMethod12.py](https://github.com/rootm0s/WinPwnage/blob/master/winpwnage/functions/uac/uacMethod12.py)

### WinPwnage usage:
`python winpwnage.py --use uac --id 12 --payload c:\\windows\\system32\\cmd.exe`
