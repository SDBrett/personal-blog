---
author: Brett Johnson
categories:
- UCSMSDK
date: "2018-03-23T00:00:00Z"
description: First look at the Python SDK for USC Manager
tags:
- Python
- Cisco
- UCS
title: Intro to UCSMSDK
---

Cisco UCSMSDK Intro

UCSMSDK is a Python SDK from Cisco for the configuration of UCS managers. The modules are supports in both Python 2 and 3, my testing has been with Python 3.6 and everything has been going well.

[GitHub Repo](https://github.com/CiscoUcs/ucsmsdk)


#### Why Use UCSMSDK

Cisco has a number of CLI options to interact with UCS Manager, depending on you circumstances some are more appropriate than others. In the work I have been doing UCSMSDK was a better fit.

PowerTool was not a good fit as local development environment is a Mac and Cisco only provides PowerTool as an MSI installer.

The USCM Ansible modules where considered however, key features such as service profiles were not implemented.

Python is readily available across platform and staff within the BU I'm working within have skillsets in this language.

Installation

The SDK is installed either through pip or the by cloning the github repo and building.

{{< highlight python >}}
pip install ucsmsdk
{{< / highlight >}}

{{< highlight bash >}}
git clone https://github.com/CiscoUcs/ucsmsdk.git
cd ucsmsdk
make install
{{< / highlight >}}

#### Documentation

The modules are well documented in that a lot of data is available. But, it can be difficult to navigate and find which module you want to use.

[Documentation link](https://ciscoucs.github.io/ucsmsdk_docs/)

The github repository contains a large number of samples which will cover many use cases. These are an excellent starting place to understand how to consume the SDK.

#### Consuming the SDK

The first step to using the SDK is to connect to a USC manager using the object UscHandle.

{{< highlight python >}}
from ucsmsdk.ucshandle import UcsHandle
# Create a connection handle
handle = UcsHandle("192.168.1.1", "admin", "password")
# Login to the server
handle.login()
{{< / highlight >}}

Interaction with the UCS Manager is achieved through the use of managed objects. Objects have attributes with values to modify. UscHandle is used to stage and commit the change. 

From the above connection example handle is the UcsHandle object connected to our UCS Manager. 

Methods:
Create: handle.add_mo()
Retrieve: handle.query_dn(), handle.query_classid(), handle .query_dns(), handle .query_classids()
Update: handle.set_mo()
Delete: handle.delete_mo()

If I wanted to add an IP block to an existing IP pool, I would use the following
{{< highlight python >}}
#Retrieve IP Pool
mo = handle.query_dn("org-root/ip-pool-ext-mgmt")

#Create IP Block within Pool
mo_1 = IppoolBlock(parent_mo_or_dn=mo, r_from="192.168.1.1", to="192.168.1.50", subnet="255.255.255.0", def_gw="192.168.1.51", prim_dns="8.8.8.8")
handle.add_mo(mo_1)
handle.commit()
{{< / highlight >}}

#### Challenges

One of the main challenges faced is finding the module to import for your specific tasks. The first place to look is within the samples as there's a good change that it'll be in there.

When the samples do not provide the information needed, I perform the action required using the UCS HTML UI and use Postman to capture the API calls.

Below is the body from the action of creating a LAN Convexity policy with vNICs assigned.
{{< highlight xml >}}
<configConfMos cookie="1521278200/5847a9e1-afe0-4ad5-8c8f-9d63feeecb75" inHierarchical="false">
    <inConfigs>
        <pair key="org-root/lan-conn-pol-vcondemo">
            <vnicLanConnPolicy name="vcondemo" dn="org-root/lan-conn-pol-vcondemo" status="created" sacl="addchild,del,mod">
                <vnicEther adaptorProfileName="VMWare" name="vnic0" nwTemplName="eth0" order="1" rn="ether-vnic0" status="created" sacl="addchild,del,mod"></vnicEther>
            </vnicLanConnPolicy>
        </pair>
    </inConfigs>
</configConfMos>
{{< / highlight >}}

Creating the LAN Connection Policy
{{< highlight xml >}}
<vnicLanConnPolicy name="vcondemo" dn="org-root/lan-conn-pol-vcondemo" status="created" sacl="addchild,del,mod">
{{< / highlight >}}

The element name from the line above shows that you need to use the module "vnicLanConnPolicy" to create the policy.
Searching the docs will lead you to the class "ucsmsdk.mometa.vnic.VnicLanConnPolicy.VnicLanConnPolicy"

To import 
{{< highlight python >}}
from ucsmsdk.mometa.vnic.VnicLanConnPolicy import VnicLanConnPolicy
{{< / highlight >}}

To add the vNIC to the policy.

{{< highlight xml >}}
<vnicEther adaptorProfileName="VMWare" name="vnic0" nwTemplName="eth0" order="1" rn="ether-vnic0" status="created" sacl="addchild,del,mod"></vnicEther>
{{< / highlight >}}

Within the vnicLanConnPolicy element there is a nested element vnicEther. Again, search the docs to find that it's the class "ucsmsdk.mometa.vnic.VnicEther.VnicEther"

To Import
{{< highlight python >}}
from ucsmsdk.mometa.vnic.VnicEther import VnicEther
{{< / highlight >}}

Putting it together gives the code
{{< highlight python >}}
#Import

from ucsmsdk.mometa.vnic.VnicLanConnPolicy import VnicLanConnPolicy
from ucsmsdk.mometa.vnic.VnicEther import VnicEther

#Create LAN Connection Policy
LanConnPolicymo = VnicLanConnPolicy(parent_mo_or_dn=parent_dn, name="esxi-host")

#Add vNIC to LAN Connection Policy
VnicEther(parent_mo_or_dn=LanConnPolicymo, name="mgmt0", nw_templ_name=mgmtVnicmo0.name,
          adaptor_profile_name="VMWare")

#Stage and commit
handle.add_mo(LanConnPolicymo, True)
handle.commit()
{{< / highlight >}}

#### Summary
The UCSMSDK is a feature rich SDK for interacting with a UCS manager. Settings within the UCS manager are classes defined as 'managed objects'

The GitHub repo has an excellent of samples which will help you get up an running with the SDK quickly. 

The documentation is data rich, but can be hard to get your head around at the start. Using a tool such as Postman to see the API calls through UI can prove valuable.
