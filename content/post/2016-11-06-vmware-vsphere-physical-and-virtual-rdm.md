---
academia_post_display_home:
- ""
author: Brett Johnson
categories:
- vSphere
date: "2016-11-06T19:54:15Z"
id: 464
tags:
- Physical RDM
- RDM
- Virtual RDM
- VMware
- vSphere
title: VMware vSphere Physical and Virtual RDM
url: /BrettsITBlog/2016/11/vmware-vsphere-physical-and-virtual-rdm/
---

#### RDM High-Level Cover

VMware vSphere RDM or Raw Device Mapping is the method to remove levels of abstraction from the storage presented to vSphere VMs. RDMs are used to meet use cases and could be considered a deviation from the &#8216;normal';.

When creating an RDM, there are two options; Virtual and Physical. A Virtual RDM provides some level of storage abstraction. Physical does not, the full capabilities are visible by the VM. Herein lies the reason to behind using RDM, some systems require this level of visibility.

When using RDM it is critical to weigh up the limitations, both Virtual and Physical impose constraints over VMDK. Physical has more constraints than Virtual. Follow this link to see the details on capabilities and limitations: https://kb.vmware.com/selfservice/microsites/search.do?language=en_US&cmd=displayKC&externalId=2009226

Performance used to be a consideration for the use of RDM, for example, SQL servers would be likely to have RDM. As vSphere improves this difference is less to the point of negligable. This should be tested on a chase by case basis.

VMDK files are used for descriptors, but not data storage.

#### How it works:

A LUN is created on the storage backend. Mapping of the LUN to ESXi hosts is done as per normal. If your SAN management software integrates with vSphere to auto mount the LUN as a Datastore, skip that step.

When a VM is configured to use a RDM, the selected LUN is dedicated. Instead of a -flat.vmdk containing data, residing on a VMFS filesystem. Data is written directly to the LUN.

A LUN cannot be mounted as a data store if it is to be used for RDM. The LUN is identified by it';s unique naa number. Make sure that you have clearly documented the naa numbers, it isn';t overly hard to mistake these.

The path for a naa is /vmfs/devices/disks/naa.aaaaaaaabbbbbbbbbbbbccdddddddddd. You can read up on the naa name format here. http://vmwise.com/2012/06/01/naa-ids-and-breaking-them-apart/

#### Attaching to a VM

To add an RDM to a VM, we go to Edit Settings > New RDM Disk > Add. Here we select the naa found before. During the creation process, we are asked to select the storage location, "With VM&#8221; or "Another Datastore&#8221;. The reason for this will be covered in a bit.

To take a step back for explanation purposes. When you have a new SDD for you a PC, it is blank, no filesystem, just a control board and NAND. This is the LUN after it';s been created on the SAN. There';s a controller, but no partition, no filesystem. The SSD is connected to your computer and you boot. That is where the above paragraph puts you.

Use normal OS partitioning and formatting. With an RDM, the partition is written directly to the LUN, the filesystem is written directly to the LUN as well and can be seen in vSphere.

[![File System](/assets/images/2016/11/File-System.png)]({{site.url}}/assets/images/2016/11/File-System.png)

That';s the RDM setup. It will work, but just knowing the steps doesn';t provide much insight as to how RDM';s work, more to the point why you see what you see.

#### RDM VMDK Files:

During the creation process, you need to select where the storage will be located. The wording is a bit poorly written. What you are selecting is where you would like the VMDK files stored.

If you have looked at VMDK storage, you might remember that a VMDK is a descriptor file, not where data is stored, that is in a VMDK1-Flat.VMDK file.

Even though we are using an RDM for storage, a VMDK is used to say where the real data is. There are some slight differences between the VMDK file for Virtual and Physical RDMs.

##### Virtual RDM

{{< highlight text >}}# Disk DescriptorFile
version=3
encoding="UTF-8"
CID=1ddd5c91
parentCID=ffffffff
isNativeSnapshot="no"
createType="vmfsRawDeviceMap"

# Extent description
RW 1048576000 VMFSRDM "RDM-rdm.vmdk"

# Change Tracking File
changeTrackPath="Sever_2RDM-ctk.vmdk"

# The Disk Data Base 
#DDB

ddb.adapterType = "lsilogic"
ddb.deletable = "true"
ddb.geometry.cylinders = "65270"
ddb.geometry.heads = "255"
ddb.geometry.sectors = "63"
ddb.longContentID = ""
ddb.toolsVersion = "10249"
ddb.uuid = "uuid"
ddb.virtualHWVersion = "9"{{< / highlight >}}

##### Physical RDM

{{< highlight text >}}# Disk DescriptorFile
version=1
encoding="UTF-8"
CID=6c6fb67d
parentCID=ffffffff
isNativeSnapshot="no"
createType="vmfsPassthroughRawDeviceMap"

# Extent description
RW 1048576000 VMFSRDM "5-rdmp.vmdk"

# The Disk Data Base 
#DDB

ddb.adapterType = "lsilogic"
ddb.geometry.cylinders = "65270"
ddb.geometry.heads = "255"
ddb.geometry.sectors = "63"
ddb.longContentID = ""
ddb.uuid = "6uuid"
ddb.virtualHWVersion = "11"{{< / highlight >}}

There are a few slight differences, the main one of interest though is the "createType&#8221;. On the Virtual RDM, it is "vmfsRawDeviceMap&#8221; on Physical it';s "vmfsPassthroughRawDeviceMap&#8221;. Also, there is no change tracking file for a Physical RDM.

The extent description is a used to describe the size of the drive being presented. If you check the Datastore it will be the same size. If you SSH to the host and run

{{< highlight text >}}dh -u /vmfs/volumes/DataStore/VM/*{{< / highlight >}}

The size is 0

##### Virtual RDM Files:

  * .vmdk &#8211; Descriptor file
  * -ctk.vmdk &#8211; Change Block Tracking
  * -rdm.vmdk &#8211; Extent file

##### Physical RDM Files:

  * .vmdk &#8211; Descriptor file
  * -rdmp.vmdk &#8211; Extent file
  * The absence of a ctk file means that there are no snapshots for a Physical RDM.

#### Convert from between RDM types:

The type of RDM is not set in stone. To change the type, simply power down the VM, edit settings and delete the Hard Disk that represents the RDM, choose to delete from disk. I can hear that getting your attention.

What we are really doing is saying that we would like to do is remove the VMDK files from the datastore. As these files are just used as descriptors, there is no data loss, that is still stored on the LUN. With a VMDK it';s another story, don';t mix that one up.

Go back into the VM settings and Add the RDM again, making sure to select the type you would like. The default is Physical.

Official VMware documentation for RDM is provided in the [vSphere Storage Guide](http://pubs.vmware.com/vsphere-60/topic/com.vmware.vsphere.storage.doc/GUID-8AE88758-20C1-4873-99C7-181EF9ACFA70.html)

&nbsp;