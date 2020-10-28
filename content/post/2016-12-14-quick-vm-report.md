---
academia_post_display_home:
- ""
author: Brett Johnson
categories:
- PowerCLI
date: "2016-12-14T06:36:46Z"
id: 560
image: /wp-content/uploads/2016/12/PowerCLI.png
tags:
- PowerCLI
- Script
- VMware
- vSphere
title: Quick VM Report with PowerCLI
url: /BrettsITBlog/2016/12/quick-vm-report/
---

I primarily use this PowerCLI script after performing VM migrations. The script output focuses on what changed during the migration. Specifically moving to hosts running a newer version of ESXi.

The script is simple to modify and adapt to fit your specific needs.

#### Using the Script

To make use of the script, all that you need to do is set the `$ClusterName` and `$Path` values at the beginning. However, think of this not as a &#8216;run as it is'; script. Consider, this to be a framework.

{{< highlight powershell >}} 
$ClusterName = “ENTER CLUSTER NAME”
$Path = “Enter CSV Output path”


get-cluster $ClusterName | Get-VM |
Select Name,
@{N="Host"; E={$_.vmhost}},
@{N="Datastore";E={[string]::Join(',',(Get-Datastore -Id $_.DatastoreIdList | Select -ExpandProperty Name))}},
@{N="Folder";E={$_.Folder.Name}},
@{N="Hardware";E={$_.version}},
@{N="ToolsStatus";E={$_.ExtensionData.Guest.ToolsStatus}},
@{N="ToolsVersion";E={$_.ExtensionData.Guest.ToolsVersion}} |
Export-Csv $Path
{{< / highlight >}}