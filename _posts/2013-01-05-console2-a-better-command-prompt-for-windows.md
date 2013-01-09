---
layout: post
title: "Console2: A better command prompt for Windows"
description: ""
category: 
tags: []
---
{% include JB/setup %}

I found out about [Console2](http://sourceforge.net/projects/console/) from Scott Hanselman's [blog](http://www.hanselman.com/blog/Console2ABetterWindowsCommandPrompt.aspx) and haven't looked back. On Windows, it's a great replacement for vanilla `cmd.exe`. It supports tabs and can host multiple shells, including cmd, Powershell, Msysgit and Cygwin. Even if you prefer the Windows command line over these Unix-inspired shells, you should still use Console2 to launch cmd sessions. 

Scott's blog tells you how to set up Powershell on Console2, but I use it primarily with the version of Bash that comes with standard [Git for Windows](http://git-scm.com/download/win) installer. 

Here's how I set up a new install of Console2:

0. Download and install [Git for Windows](http://git-scm.com/download/win). Preferably to `c:\bin\git`
0. Download and unzip [Console2](http://sourceforge.net/projects/console/) somewhere. Preferably to `c:\bin\console2`
0. Launch Console2
0. Go to Edit &rarr; Settings &rarr; Tabs
0. Rename the default `Console` entry to `Windows`. This points to `cmd`
0. Add a new tab
  * Title: `Git`
  * Icon: `c:\bin\git\etc\git.ico`
  * Shell: `c:\bin\git\bin\sh.exe --login -i`
  * Startup dir: `c:\`
0. If you prefer to use this as your default shell, like I do, move `Git` to the top of the list.
0. Press OK and you're done! 

I also like to edit a few other settings. These are all personal preferences:

* Open Edit &rarr; Settings
* Under Behavior, check the `Copy on Select` option
* Under Hotkeys
  * Set `New Tab 1` to `Ctrl-T`
  * Set `New Tab 2` to `Ctrl-Shift-T`
  * Set `Paste` to `Ctrl-V`

Cheers!