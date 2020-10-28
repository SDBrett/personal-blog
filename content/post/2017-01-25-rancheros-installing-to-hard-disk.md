---
academia_post_display_home:
- ""
author: Brett Johnson
categories:
- Cloud
date: "2017-01-25T11:23:01Z"
id: 609
image: /wp-content/uploads/2017/01/rancheros-logo-01.png
tags:
- Linux
- RancherOS
title: 'RancherOS: Installing to hard disk'
url: /BrettsITBlog/2017/01/rancheros-installing-to-hard-disk/
---

Recently I set about installing RancherOS. This was just to have a look and see what use cases it might help with. I chose to install RancherOS to a VM. In my case, VMware workstation. A roadblock I hit was providing an SSH key to the cloud-config.yml file.

The roadblock specifically, how can I send a file to a system I don';t have a password or SSH key for?

This article covers, generating an SSH key, SSH access to live CD and installation to hard disk.

#### Requirements and Prior reading

Before starting with RancherOS, it';s a good idea to read the documentation. This is available [here](https://docs.rancher.com/os/running-rancheros/workstation/boot-from-iso/). We will be performing a bare metal install to disk, specific documentation is [here](https://docs.rancher.com/os/running-rancheros/server/install-to-disk/).

You need to download the RancherOS ISO image. See above link.

Your VM does not require much in the way of resources. 1vCPU and at least 1.5GB of RAM. The minimum RAM requirement is 512MB. However, 1.5GB is recommended for the installation. For my testing, I have only allocated 20GB of disk space, adjust as required.

#### Booting RancherOS for the first time

Configure your VM and point to the RancherOS ISO image as boot media. The ISO image is a live boot disk and will auto login to the account rancher. We do not know the rancher password. This presents an issue with accessing via SSH. We will need to create a password. This will not come across during installation.

Run the following commands:

{{< highlight bash >}} sudo /bin/bash
passwd rancher{{< / highlight >}}

This will take you to a bash prompt as root and allow you to set a password for the rancher user account.

Set the password and use exit to go back ranchers shell.

<p class="">
  Now that the password has been set, we will SSH to the VM.
</p>

Launch your preferred terminal emulator and SSH using the rancher username and the password you just set.

#### Generate SSH keypairs

There are a number of ways to generate an SSH keypair. As I';m on a Windows system, I will be using PuTTYgen. Which can be downloaded from available [here](http://www.chiark.greenend.org.uk/~sgtatham/putty/latest.html).

Launch PuTTYgen and click the generate button. Move your mouse around the blank space to generate the key.
[![Putty Gen](/assets/images/2017/01/PuttyGen.png)]({{site.url}}/assets/images/2017/01/PuttyGen.png)

Enter a key comment, this is optional.

[![Generated](/assets/images/2017/01/PuttyGen-Generated.png)]({{site.url}}/assets/images/2017/01/PuttyGen-Generated.png)

Enter passphrase if you choose. This is optional but better security.

Save the public and private key.

Select the public key and copy.

[![Selected](/assets/images/2017/01/PuttyGen-Selected.png)]({{site.url}}/assets/images/2017/01/PuttyGen-Selected.png)

#### Create Cloud-Config.yml

Using the SSH session we opened before, we will create our cloud-config.yml file. Running the following command

{{< highlight bash >}}vi cloud-config.yml{{< / highlight >}}

Type in the relevant information for your system. I have provided my cloud-config.yml file as an example.

{{< highlight yaml >}}}
#cloud-config

hostname: rancher

rancher:
  network:
    interfaces:
      eth*:
       dhcp: false
      eth0:
       address: 192.168.195.138/24
       gateway: 192.168.195.2
    dns:
     nameservers:
      - 8.8.8.8

ssh_authorized_keys:
  - ssh-rsa AAAAB3NzaC1yc...TiYoEROEPeotRn+NkHIMLE4FcgH7qF1Q== RancherOS
{{< / highlight >}}

When entering the ssh\_authorized\_keys past in the public key from PuTTYGen that you copied before.

Write quit vi.

You should now have a valid cloud-config.yml file with an SSH key that matches the private key on your system.

#### Install RancherOS to disk

Installing RancherOS to disk is a single line command. If your VM does not have internet access, go back to the documentation link. Rancher provides details on that method.

{{< highlight bash >}}sudo ros install -c cloud-config.yml -d /dev/sda{{< / highlight >}}

After the installation, reboot the system.

#### SSH to Installed OS

Just to recap, we now have RancherOS installed to disk and a valid key pair for the purpose of SSH.

Using your preferred SSH client, connect using the username rancher and the private key you saved from PuTTYgen.

Putty:

Enter IP Address and Name for Saved Session as we will save this.

[![Enter IP](/assets/images/2017/01/Putty-Rancher-1.png)]({{site.url}}/assets/images/2017/01/Putty-Rancher-1.png)

Go to Connection > Data and enter the auto-login name as rancher

[![Auto login](/assets/images/2017/01/Putty-Rancher-2.png)]({{site.url}}/assets/images/2017/01/Putty-Rancher-2.png)

Go to Connection > SSH > Auth and browse for the Private Key

[![Browse](/assets/images/2017/01/Putty-Rancher-3.png)]({{site.url}}/assets/images/2017/01/Putty-Rancher-3.png)


Go back to Sessions and click Save.

MobaXterm (Another Terminal Emulator)

[![Logged In](/assets/images/2017/01/MobaXterm-Rancher.png)]({{site.url}}/assets/images/2017/01/MobaXterm-Rancher.png)

##### Summary

Initially, I had trouble being able to ssh to the live session. Which meant I could not create the cloud-config.yml file with a valid SSH key. These are the steps I used to get everything setup for testing.