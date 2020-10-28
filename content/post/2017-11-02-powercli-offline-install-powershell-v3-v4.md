---
author: Brett Johnson
categories:
- PowerCLI
date: "2017-11-02T11:25:25Z"
tags:
- PowerCLI
- PowerShell
- VMware
title: 'PowerCLI: Offline install PowerShell v3 and v4'
url: /BrettsITBlog/2017/11/powercli-offline-install-powershell-v3-v4/
---


Today I came across the need to use PowerCLI on a computer without internet access. Thankfully I found this [blog post](https://blogs.vmware.com/PowerCLI/2017/04/powercli-install-process-powershell-gallery.html) detailing the steps that I needed to perform, however not all was rainbows and unicorns. I quickly ran aground when attempting to import the modules. After a bit of research, I learnt that Microsoft had changed the module directory structure in PowerShell v5. This new structure is not compatible with previous versions.

#### **The Problem**

When using `Install-Module` to install a module, the module files as located in a subdirectory which is the version number. This is not a supported directory structure for PowerShell version 3 and 4. This caused an error when attempting to import, PowerShell could not find the modules.

[![PowerShell-v5-Module-Structure](/assets/images/2017/11/PowerShell-v5-Module-Structure.png)]({{site.url}}/assets/images/2017/11/PowerShell-v5-Module-Structure.png)

[![PowerShell-v5-Module-Structure](/assets/images/2017/11/PowerShell-v3v4-Module-Structure.png)]({{site.url}}/assets/images/2017/11/PowerShell-v3v4-Module-Structure.png)

#### The Solution

The solution to the problem is simple enough, move the module files up one directory. As there are a number of modules this can be tedious. So here';s a script to do that for you

{{< highlight powershell >}}
$ModuleFolders = Get-ChildItem VMware*

Foreach($ModuleFolder in $ModuleFolders){
    $VersionFolder = $ModuleFolder | Get-ChildItem -Directory
    $CopySourceString = $VersionFolder.FullName + "\*"
    Copy-Item $CopySourceString $ModuleFolder.FullName -Verbose
}
{{< / highlight >}}

Further details on the script are available on [GitHub](https://github.com/SDBrett/powershellScripts/tree/master/powerCLIOfflineInstallFix).

After running the script, I was able to user PowerCLI for super-secret spy business.

&nbsp;