---
author: Brett Johnson
categories:
- Server 2012
date: "2016-08-09T21:36:34Z"
id: 320
tags:
- Microsoft
- Windows
title: MS Local Administrator Password Solution. Part 2
url: /BrettsITBlog/2016/08/ms-local-administrator-password-solution-part-2/
---
In [part 1](https://sdbrett.com/BrettsITBlog/2016/06/ms-local-administrator-password-solution-part-1/), we looked at making the necessary changes to AD for LAPS, from extended the schema to modifying the object attribute security.

In this part, we will go through deploying the LAPS agent on a workstation. This process is very straight forward, we will use GPO to deploy the agent to our workstation and confirm that the password is now random and stored in AD.

During the configuration of the workstation, I set the admin password as "Password1&#8221;, secure I know.

MS provide two separate installers, one for 32Bit and one for 64Bit. I suggest you download both.

On my DC I have created a share called "LapsDeploy&#8221;. Share permissions are &#8216;bypassed'; by using Everyone Full Access and the NTFS permissions have been configured to allow Authenticated Users Read and Execute permissions.

On the DC, in GPMC I have created a policy called "LapsDeploy&#8221; and linked it to the LapsComputers OU, which is where my test workstation sits.

[![LAPS GPO](/assets/images/2016/08/Laps-GPO-Link.png)]({{site.url}}/assets/images/2016/08/Laps-GPO-Link.png)

I';m not going to cover the individual steps for software deployment via GPO.

In GPO there are two sections that need to be configured. There is software deployment and the second is to allow password management, as well as some complexity settings.

As there are separate installers for 32 and 64 bit operating systems, it';s important to open the 32-bit installer properties, go to Deployment and Advanced. Make sure that "Make this 32-bit x86 application available to Win64 machines&#8221; is unticked.

[![LAPS GPO x86](/assets/images/2016/08/Laps-GPO-x86.png)]({{site.url}}/assets/images/2016/08/Laps-GPO-x86.png)

After the software installation part is done, navigate to Policies > Administrative Templates > LAPS. At a bare minimum the setting "Enable local admin password management&#8221; needs to be enabled. The rest are optional.

[![LAPS Settings](/assets/images/2016/08/Laps-Setting-Management.png)]({{site.url}}/assets/images/2016/08/Laps-Setting-Management.png)

Computers affected by the OU need to be restarted for the software to be installed. Sometimes two restarts are required.

Checking Programs and Features shows that LAPs is now installed.

[![LAPS Installed](/assets/images/2016/08/Laps-installed.png)]({{site.url}}/assets/images/2016/08/Laps-installed.png)

The System event logs also show that installation was successful.

[![Event Logs](/assets/images/2016/08/Laps-Eventviewer.png)]({{site.url}}/assets/images/2016/08/Laps-Eventviewer.png)

Back on the DC we can check that the password has been set using the fat client "LAPS UI&#8221; or through PowerShell.

[![LAPS Fat Client](/assets/images/2016/08/Laps-fat-client.png)]({{site.url}}/assets/images/2016/08/Laps-fat-client.png)

[![LAPS Password](/assets/images/2016/08/Laps-Password.png)]({{site.url}}/assets/images/2016/08/Laps-Password.png)

That's it, LAPs is deployed and working.

Personally, I have found LAPS to be a great tool and easy to deploy and thinks it';s under-utilized in the real world.