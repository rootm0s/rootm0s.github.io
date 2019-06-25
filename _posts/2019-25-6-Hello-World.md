---
layout: post
title: UIACCESS UAC BYPASS
---

In these examples, we start a host process (msra.exe) that we steal the UIAccess token from. We downgrade the token IL from Medium+ to Medium. We use the token to spawn a new process (uihack.exe) with the UIAccess flag, we can now send keyboard events to the elevated processes.

Not designed to be stealthy, but it's for sure possible! This is a demo in Python, just to display how it works.

You need to build the uihack python file to an executable, make sure it stays in dist folder. Once you created the uihack executable, you can launch uiap.py from a non-elevated command prompt.

Here's a few methods, showing how hijacking UIAccess tokens can be used to bypass UAC.

Rstrui method:

This executable is running elevated by default. Since rstrui executable is vulnerable to class hijacking, we use that to spawn our executable which is defined inside uihack file. In order to trigger the hijack, we need to send keyboard events to the window. Upon success, a elevated console window or custom executable should appear.

Taskmgr method:

This executable is running elevated by default, we send keyboard events to the window in order to launch an elevated console using the "Run new task" feature in Task Manager. Upon success, a elevated console window should appear.

Msconfig method:

This executable is running elevated by default, we send keyboard events to the window in order to navigate through the list of available tools until we reach Command Prompt. Upon success, a elevated console window should appear.

![_config.yml]({{ site.baseurl }}/images/config.png)

The easiest way to make your first post is to edit this one. Go into /_posts/ and update the Hello World markdown file. For more instructions head over to the [Jekyll Now repository](https://github.com/barryclark/jekyll-now) on GitHub.
