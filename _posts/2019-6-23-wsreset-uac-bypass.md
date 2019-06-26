---
layout: post
title: WSReset UAC bypass
---

This executable is running elevated by default. Since WSReset executable is vulnerable to
class hijacking, it can be used to spawn our executable with High IL. We need to hijack the default
value at "HKCU\Software\Classes\AppX82a6gwre4fdg3bt635tn5ctqjf8msdd2\Shell\open\command" and create a new string value with the
name DelegateExecute set to 0.

Tested on Windows 10 (17134)

### Full code at WinPwnage project:
[https://github.com/rootm0s/WinPwnage/blob/master/winpwnage/functions/uac/uac_wsreset.py](https://github.com/rootm0s/WinPwnage/blob/master/winpwnage/functions/uac/uac_wsreset.py)

### WinPwnage usage:
`python winpwnage.py --use uac --id 20 --payload c:\\windows\\system32\\cmd.exe`
