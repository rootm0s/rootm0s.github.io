---
layout: post
title: SDCLT UAC BYPASS (App Paths)
---

This executable is running elevated by default. Since sdclt executable is vulnerable to
registry App Path hijacking, it can be used to spawn our executable with High IL. We need to hijack the default
value at "HKCU\Software\Microsoft\Windows\CurrentVersion\\App Paths\control.exe" with path to executable to be launched.

This works from Windows 10 (10240) to Windows 10 (16215)

### Full code at WinPwnage project:
[https://github.com/rootm0s/WinPwnage/blob/master/winpwnage/functions/uac/uac_sdcltcontrol.py](https://github.com/rootm0s/WinPwnage/blob/master/winpwnage/functions/uac/uac_sdcltcontrol.py)

### WinPwnage usage:
`python winpwnage.py --use uac --id 6 --payload c:\\windows\\system32\\cmd.exe`
