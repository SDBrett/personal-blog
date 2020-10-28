---
author: Brett Johnson
categories:
- PowerShell
date: "2015-02-11T04:46:37Z"
id: 48
tags:
- Server 2012
title: Default to PowerShell in Windows Core
url: /BrettsITBlog/2015/02/default-to-powershell-in-windows-core/
---
One of the big pushes in Server 2012 is the use a Server Core installation instead of the full GUI install. Microsoft have put a lot of effort into encouraging administrators to use PowerShell as a core tool for day to day administration of servers, which is why is seems a bit strange that a Server Core installation boots to a traditional command prompt instead of a PowerShell prompt.

To change your Server Core installation to launch PowerShell instead of the normal Command Prompt is a simple registry change.

From the Command Prompt run _Regedit_

Navigate to _HKLM\Software\Microsoft\Windows NT\CurrentVersion\Winlogon_

![Regedit]({{ "/assets/images/2015/02/PowerShell-reg-default.png" | absolute_url }})

The highlighted _Shell_ dword is what we need to change, so double click on that and type in _powershell.exe_

![New Key]({{ "/assets/images/2015/02/updated.png" | absolute_url }})

Click _Ok_ and close the Registry Editior.

When you reboot the server you will now have a PowerShell prompt.