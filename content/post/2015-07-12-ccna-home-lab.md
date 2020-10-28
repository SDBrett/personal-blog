---
author: Brett Johnson
categories:
- CCNA
date: "2015-07-12T22:22:06Z"
id: 51
title: CCNA Home LAB
url: /BrettsITBlog/2015/07/ccna-home-lab/
---
During my study for my VCP5-DCV certification I found that having a home lab was a huge asset in terms of being able to get a better understanding of the subject matter. Now that I am working towards my CCNA Routing and Switch I feel this will be the case once again.

My goal is use the home lab to work on labs as per the study guides I am using and also to have something productive to work on when I don';t feel like reading through a study guide. Let';s face it all study guides can get a bit dry.

The home lab PC was originally spec';d up for running a nested ESXi environment and is overkill for what I will be doing for my CCNA.

  * i5-4690
  * 32 GB Ram
  * 512 GB SSD drive
  * Windows 8.1

To the setup.

I will be using GNS3 and VirtualBox to setup the lab environment, which is a pretty standard setup for people undertaking the CCNA. VirtualBox is a requirement to be able to run IOU devices as well as being able to connect VMs into the network.

I have started my configuration a bit out of order from the study guide I am using so I can test connectivity. The only changes to the router from a blank config are the setup of DHCP server, IP configuration of FastEthernet 0/0 and 0/1 as well has no shutdown.

**Starting Topology**

The starting topology is very simple, a router R1 and Layer2 device through IOU and a Debian VM.

![Topology]({{ "/assets/images/2015/07/Network-topology.png" | absolute_url }})

**The DHCP config**

{{< highlight plaintext >}}

no ip dhcp use vrf connected
 ip dhcp excluded-address 192.168.10.1 192.168.10.20
 !
 ip dhcp pool Internal
 network 192.168.10.0 255.255.255.0
 dns-server 4.4.4.4
 default-router 192.168.10.1
{{< / highlight >}}


**Interface Setup**

{{< highlight plaintext >}}

interface FastEthernet0/0
ip address 192.168.10.1 255.255.255.0
duplex auto
speed auto
!
interface FastEthernet0/1
ip address 192.168.20.1 255.255.255.0
duplex auto
speed auto
!
{{< / highlight >}}


Getting communication between the Debian VM and the devices took a bit of effort, but wasn';t too bad.

Adding a VM to GNS3 is a very simple process. Go to Edit > Preferences > VirtualBox VMs > New. Then just select the VM you want to add and your done.

To get the VM to communicate with the devices go to Edit > Preferences > VirtualBox VMs > Edit > Network and tick Allow GNS3 to use any configured VirtualBox Adapter

![Network]({{ "/assets/images/2015/07/debian-network-GNS3.png" | absolute_url }})

When I powered on the project the network settings for the VM in VirtualBox had been updated.

![Network Adapter]({{ "/assets/images/2015/07/debian-network-vb.png" | absolute_url }})


Everything booted fine and communication tested well.

![ifconfig]({{ "/assets/images/2015/07/ifconfig.png" | absolute_url }})

![Ping]({{ "/assets/images/2015/07/host-to-router-ping.png" | absolute_url }})

![Ping]({{ "/assets/images/2015/07/host-to-router-fast-01.png" | absolute_url }})

Communication from VM to router has been confirmed and that the VM can communicate with FastEthernet 0/1 which is on another network.

&nbsp;

This is my starting configuration for my CCNA lab which I';m sure will get plenty of use as well has get broken many times during the course of my study.

&nbsp;

**Study material:**

[Sybex CCNA Routing and Switching](http://www.amazon.com.au/Routing-Switching-Deluxe-Study-Guide-ebook/dp/B00R04DDK8/ref=sr_1_1?ie=UTF8&qid=1436703124&sr=8-1&keywords=sybex+ccna "Sybex CCNA Routing and Switching")

[Chris Bryant';s CCNA Study Guide, Volume 1](http://www.amazon.com.au/Chris-Bryants-CCNA-Study-Guide-ebook/dp/B00GFYEZ1A/ref=sr_1_5?ie=UTF8&qid=1436703267&sr=8-5&keywords=ccna "Chris Bryant's CCNA Study Guide, Volume 1")

[Chris Bryant';s CCNA Study Guide, Volume 2](http://www.amazon.com.au/Chris-Bryants-Study-Guide-Volume-ebook/dp/B00H9ICMV6/ref=pd_sim_351_1?ie=UTF8&refRID=0JSBJ969VBMDVVEGR3FY "Chris Bryant's CCNA Study Guide, Volume 2")

[CBTNuggets](https://www.cbtnuggets.com/ "CBT Nuggets")