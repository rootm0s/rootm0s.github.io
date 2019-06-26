---
layout: post
title: SDCLT UAC BYPASS
---

This executable is running elevated by default. Since sdclt executable is vulnerable to
class hijacking, it can be used to spawn our executable with High IL. We need to hijack the default value at "HKCU\Software\Classes\Folder\shell\open\command" with full path to our executable and create a new string value with the name DelegateExecute set to 0.

This works from Windows 10 (14393) to *Unfixed*

### Full code at WinPwnage project:
[https://github.com/rootm0s/WinPwnage/blob/master/winpwnage/functions/uac/uac_sdclt.py](https://github.com/rootm0s/WinPwnage/blob/master/winpwnage/functions/uac/uac_sdclt.py)

### WinPwnage usage:
`python winpwnage.py --use uac --id 16 --payload c:\\windows\\system32\\cmd.exe`
