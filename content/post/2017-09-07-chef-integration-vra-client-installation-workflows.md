---
author: Brett Johnson
categories:
- Chef
- vRA
date: "2017-09-07T21:49:12Z"
tags:
- Chef
- VMware
- vRA
- vRO
title: 'CHEF: Integration with vRA, Client installation workflows'
url: /BrettsITBlog/2017/09/chef-integration-vra-client-installation-workflows/
---

Chef Integration with vRA: [Part1](https://sdbrett.com/BrettsITBlog/2017/09/chef-integration-vra-installing-vro-plugin/)

In the previous Chef integration with vRA post, we covered how to install the Chef plugin for vRO. In this post, we will look at provided workflows used to install the Chef Client on Windows and Linux VMs. The workflows are called by the Event Broker Servers (EBS) after a VM is provisioned.

#### The Client Provision Workflow

To begin launch the vRO client and navigate to the Chef workflow folder. From here drill down to vRA Extensibility Samples > vRA 7 EBS. Within this folder there is a workflow called “EBS – Machine Provisioned – Chef”.

[![Workflow Tree](/assets/images/2017/09/vRO-Worflow-List.png)]({{site.url}}/assets/images/2017/09/vRO-Worflow-List.png)

This is the workflow the will be called by the EBS after the machine is provisioned and registered as “Turned on” with vRA. You may need to make changes to this workflow or some of the nested workflows it contains. As such, I would recommend duplicating the workflows to another folder to preserve the originals.

[![Client Schema](/assets/images/2017/09/Omnibus-Linux-Client-Install-Schema.png)]({{site.url}}/assets/images/2017/09/Omnibus-Linux-Client-Install-Schema.png)

Looking at the attributes, there are two of interest, _guestUsername_ and _guestPassword_. The presence of these two workflows indicate that we will need to pass credentials from vRA. As this workflow runs after provision, you need to consider how those accounts will be placed on the VM.

[![Provisioned Attributes](/assets/images/2017/09/EBS-Provisioned-Attributes.png)]({{site.url}}/assets/images/2017/09/EBS-Provisioned-Attributes.png)

For Linux, there is an option to install the client through SSH, which allows a key file to be used instead of a password.

A couple of options are:

  * The account exists on the base VM
  * The VM is configured to support external identity sources prior to the client running
  * The user can create these with the form input, this workflow runs before the client install

The payload input is a standard input for workflows called by EBS. The contents of the payload will vary based on the event topic. Don’t modify this input field.

[![Provisioned Inputs](/assets/images/2017/09/EBS-Provisioned-Input.png)]({{site.url}}/assets/images/2017/09/EBS-Provisioned-Input.png)

There are several steps to the workflow, but I want to cover the client installation ones as these are the most likely to require modification to suite.

#### Linux Client Install Workflow

[![VMtools Install Schema](/assets/images/2017/09/VMtools-Linux-Client-Install-Schema.png)]({{site.url}}/assets/images/2017/09/VMtools-Linux-Client-Install-Schema.png)

The workflow used to install the Chef client on a Linux VM attempts to retrieve a file called “install.sh” from the external web server. This script is then run on the Linux host, utilizing a Chef project called Omnitruck. Omnitruck determines the correct installation package and command to install based on the Linux distro.

[![Install attributes](/assets/images/2017/09/VMtools-Linux-Client-Install-Attributes.png)]({{site.url}}/assets/images/2017/09/VMtools-Linux-Client-Install-Attributes.png)

There are a couple of key points to know:

  * The default method is to use and external source to retrieve the installation package.
  * By default, it will get the latest version (YOLO mode)

You can modify the attribute “_command_” to store the install.sh file locally and specify the client version you require. This is a sensible thing to do. Additionally, you can change the script to just run a command for the installation.

In my lab environment, my VMs do not have internet access. Thus, I have an apache server hosting the install script and installation package. The only changes I needed to make to the original workflow was updating “command” to point to my web server. The install.sh script was updated to suit the environment.

{{< highlight bash >}}
wget  http://labrepo.sdbrett.lab/chef/chef-13.3.42-1.el7.x86_64.rpm -O /tmp/chef-13.3.42-1.el7.x86_64.rpm
rpm -Uvh /tmp/chef-13.3.42-1.el7.x86_64.rpm
{{< / highlight >}}

[![Web Server](/assets/images/2017/09/web-server.png)]({{site.url}}/assets/images/2017/09/web-server.png)

#### Windows Client Install Workflow

[![Windows Schema](/assets/images/2017/09/VMtools-Windows-Install-Schema.png)]({{site.url}}/assets/images/2017/09/VMtools-Windows-Install-Schema.png)

The Windows installation workflow uses vRA property groups to determine the location of the installer. The location is not set as part of the workflow attributes. We will look at the property groups in the an upcoming post.

Depending on the security configuration of Windows VMs, you may find that UAC doesn’t play nicely when it comes to running the installer. This is due to the default workflow using “Run Script In Guest” which utilizes VM Tools to run scripts. The downside to this, is that there is no handling of UAC.

Chef have included another option to install the client on a Windows VM, a workflow called “Windows PS Install Chef-Client”.  This workflow uses a PowerShell host to run commands on the new VM through WinRM. This can help with working around UAC. Using a PowerShell host does require the installation of another VM.

When deploying to a Windows VM, you must make considerations around how you handle UAC.

#### Summary

The Chef plugin provides two workflows for each Linux and Windows to install the client. Both have considerations that need to be made prior to commencing. If you’re vRA environment has an enterprise licence, you could use vRA “Software Components” to install the client.

The workflows covered in this post focus on vSphere VM deployments. If you would like to install the Chef client on other provisioned machines, you will need to create your own workflows.