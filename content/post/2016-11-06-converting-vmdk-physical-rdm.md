---
academia_post_display_home:
- ""
author: Brett Johnson
categories:
- vSphere
date: "2016-11-06T17:26:56Z"
id: 455
tags:
- RDM
- VMware
- vSphere
title: Converting VMDK to Physical RDM
url: /BrettsITBlog/2016/11/converting-vmdk-physical-rdm/
---

I';ve been having some issues with the conversion of a VMDK to a Physical RDM. This is a request from a client, as the storage and backup vendor has stated that there are advantages to doing this. Not something I have done before, but nothing wrong with that.

I found the KB article with ease: <https://kb.vmware.com/selfservice/microsites/search.do?language=en_US&cmd=displayKC&externalId=3443266>

Steps looked straight forward, get the identifier and power down the server, run the commands. Yeah, Nah.

At this point, I';m going to take the assumption that you already understand how to create an RDM for a VM using the VM settings.

#### Stop&#8230; Error Time, can';t complete this!!!!

<span style="color: #ff0000;">Destination disk format: pass-through raw disk mapping to &#8216;/vmfs/devices/disks/naa.6xxxxxxxxxxxxxxxx';</span>
  
<span style="color: #ff0000;">Cloning disk &#8216;/vmfs/volumes/DS1/VM/VM.vmdk';&#8230;</span>
  
<span style="color: #ff0000;">Failed to clone disk: One of the parameters supplied is invalid (1).</span>

#### Findings:

I first found that there are a number of articles referring to the conversion of RDM to VMDK and Physical RDM to Virtual. Unfortunately, articles and posts about the process I am attempting are around vSphere 3.5. Back then the command structure was a lot different and I couldn';t adapt that.

#### The problem:

According to the VMware article, there are two commands to run for the conversion, depending on if you want to convert to Physical RDM or Virtual RDM

Virtual

{{< highlight text >}}vmkfstools –i srcfile -d rdm:/vmfs/devices/disks/identifier /vmfs/volumes/datastore/vmdir/vmname.vmdk{{< / highlight >}}

Physical

{{< highlight text >}}vmkfstools –i srcfile -d rdmd:/vmfs/devices/disks/identifier /vmfs/volumes/datastore/vmdir/vmname.vmdk{{< / highlight >}}

The difference in the commands is that Virtual uses "rdm&#8221; and physical uses "rdmp&#8221;.

However, using rdmp gave the error above. After a good period of time and\ oOn the brink of a bottle-o run, I looked into converting virtual RDM to physical RDM, as well as digging more around RDM in general. Maybe I could convert VMDK > Virtual RDM, then Virtual RDM > Physical RDM.

#### Solution:

I found that converting Virtual RDM to Physical RDM is easier then finding bad coffee at Starbucks.

Refer: <https://kb.vmware.com/selfservice/microsites/search.do?language=en_US&cmd=displayKC&externalId=1006599>

I ran the conversion using rdm instead of rdmp. After that, I removed the VMDK files that it generated.

Edit VM Settings > Add > RDM Disk > Select device used in the conversion command. The default mount setting is physical.

To verify I run

{{< highlight powershell >}}get-vm VMNAME | Get-HardDisk -DiskType RawPhysical{{< / highlight >}}

{{< highlight text >}}CapacityGB Persistence Filename
---------- ----------- --------
500.000 IndependentPersis... [DS] VM/VM1_4.vmdk
500.000 IndependentPersis... [DS] VM/VM1_5.vmdk{{< / highlight >}}

After booting the VM was the final test and everything appears good. The drive mappings in Windows also remained correct which was nice.

All is good in the land of OZ and I hope this gives the client the benefits that they have been told it will.