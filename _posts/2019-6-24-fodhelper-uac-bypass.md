---
layout: post
title: Fodhelper UAC bypass
---

This executable is running elevated by default. Since fodhelper executable is vulnerable to
class hijacking, it can be used to spawn our executable with High IL. We need to hijack the default
value at "HKCU\Software\Classes\ms-settings\shell\open\command" and create a new string value with the
name DelegateExecute set to 0.

This works from Windows 10 (10240) to *Unfixed*

### Full code at WinPwnage project:
[https://github.com/rootm0s/WinPwnage/blob/master/winpwnage/functions/uac/uac_fodhelper.py](https://github.com/rootm0s/WinPwnage/blob/master/winpwnage/functions/uac/uac_fodhelper.py)

### WinPwnage usage:
´python winpwnage.py --use uac --id 2 --payload c:\\windows\\system32\\cmd.exe´
