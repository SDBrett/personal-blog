---
author: Brett Johnson
categories:
- Vmware
date: "2017-11-03T07:42:49Z"
tags:
- HowTo
- Linux
- VMware
title: VMware Workstation 14 Linux Not Enough Memory
url: /BrettsITBlog/2017/11/vmware-workstation-14-linux-not-enough-memory/
---

I recently updated my Ubuntu install from 17.04 to 17.10, which meant the kernel was updated to 4.13. As a result, I was no longer able to run VMs with VMware Workstation.

I found some posts regarding the error and that it was due to a change in the way the Linux kernel handles paging. To resolve the issue I needed to replace the vmmon modules.

I didn';t take down the entire error message, but [this post](https://superuser.com/questions/1255099/vmware-workstation-not-enough-physical-memory-since-last-update) references the same error. There are a number of other posts on the VMware community forums for the same error.

#### The Fix

First of all, please go to the [GitHub repo](https://github.com/mkubecek/vmware-host-modules/tree/workstation-14.0.0) and read the disclaimers in the README.MD.

The repo has a number of branches, covering different versions of Workstation and VMware player. Adjust the `git clone` command to suit your version by replacing `workstation-14.0.0` with a version relevant to you.

Run the below commands and the memory issue should be resolved

{{< highlight bash >}}
#Clone workstation 14 branch
git clone -b workstation-14.0.0 https://github.com/mkubecek/vmware-host-modules.git
cd vmware-host-modules
#Overwrite existing vmmon archive
sudo tar cfv /usr/lib/vmware/modules/source/vmmon.tar vmmon-only/
#Install new modules
sudo vmware-modconfig --console --install-all
{{< / highlight >}}