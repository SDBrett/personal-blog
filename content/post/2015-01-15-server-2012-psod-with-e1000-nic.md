---
author: Brett Johnson
categories:
- Vmware
date: "2015-01-15T21:06:36Z"
id: 23
title: Server 2012 PSOD With E1000 NIC
url: /BrettsITBlog/2015/01/server-2012-psod-with-e1000-nic/
---
While testing out a Server 2012 install on ESXi 5.5 I noticed that when copying files between it and any other server I would get a PSOD.

The main section of the PSOD that got my attention was `E1000PollRxRring@vmkernel` 

After a bit of searching around I found that there seems to be a common issue with Server 2012 and Server 2012 R2 using the E1000 NIC. Standard solution seems to be remove your current E1000 NIC from the guest machine and add a NIC running as VMXNET3.

My steps for installing a server 2012 based OS as a VM on ESXi are:

  * Create VM with settings as needed, leaving the NIC as E1000
  * Install the guest OS
  * Install VM Tools
  * Shutdown the guest
  * Remove the E1000 NIC and add a new NIC as VMXNET3 .