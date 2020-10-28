---
author: Brett Johnson
categories:
- Chef
- vRA
date: "2017-09-25T15:59:20Z"
tags:
- Chef
- VMware
- vRA
- vRO
title: 'CHEF: vRA Integration, Property Groups and Blueprints'
url: /BrettsITBlog/2017/09/chef-vra-integration-property-groups-and-blueprints/
---

The Chef plugin provides some workflows to help with getting the vRA integration up and running with minimal effort. I would suggest treating them as samples to build functionality on top of. There are some limitations from the default state which might not provide sufficient flexibility for production usage.

#### Setting up the Property Groups:

Launch the vRO Client and navigate to the Chef workflow ‘_Create Property Group for Chef_ EBS _Workflows_’.

[![vRO Chef Client](/assets/images/2017/09/vCO-Chef-client-tree.png)]({{site.url}}/assets/images/2017/09/vCO-Chef-client-tree.png)

Run the workflow and enter the required information

[![Property Groups](/assets/images/2017/09/vRO-Chef-Client-Add-Prop-Groups.png)]({{site.url}}/assets/images/2017/09/vRO-Chef-Client-Add-Prop-Groups.png)

  * Property Group Name 
      * Unique name to identify the property group in vRA
  * Chef Runlist 
      * Array of run lists to be run as part of the first run
  * Install Chef Client on VM’s 
      * This Boolean option determines if the Chef client is to be installed
  * Windows MSI Installer URL 
      * While this entry doesn’t have isn’t relevant to Linux VMs, the field is mandatory.
  * Install As service 
      * Again, only relevant to Windows, but is optional
  * Guest Username and Password 
      * Enter credentials that will be available at the time of deployment
      * If you’re using guest customization scripts to create accounts, these will be created before the POST Machine Provisioning event which installs the Chef client.

Click Submit and wait for the workflow to complete.

#### Checking Property Group in vRA

Login into your vRA tenant and go to **Administration** > **Property Dictionary** > **Property** **Groups**.

Here you will see the new group that you just created. If you like, you can remove options where are not required, such as the MSI path if it’s not required.

[![Property Groups](/assets/images/2017/09/vRA-Chef-client-property-groups.png)]({{site.url}}/assets/images/2017/09/vRA-Chef-client-property-groups.png)

#### Adding Property Group to a Blueprint

In vRA navigate to **Design** > **Blueprints** and select an existing IaaS blueprint or create a new one. The blueprint should deploy a Virtual Machine, I have only tested this with a vSphere VM, no other deployments yet.

[![IaaS Blueprint](/assets/images/2017/09/vRA-IaaS-Blueprint-VM.png)]({{site.url}}/assets/images/2017/09/vRA-IaaS-Blueprint-VM.png)

From the blueprint, select the virtual machine and go to properties and Click **Add** then select your Property Group. You can view the properties that have been imported through the button ‘_View Merged Properties_’ at the bottom left.

[![Merged Properties](/assets/images/2017/09/vRA-Blueprint-merged-properties.png)]({{site.url}}/assets/images/2017/09/vRA-Blueprint-merged-properties.png)

**Save** and Close the blueprint. At this point, I will assume that you have gone through the catalog deployment steps.

#### Configuring EBS

To make the Install Chef Client workflow fire when a Virtual Machine is deployed, we need to configure an EBS subscription. The subscription will run when an event matching the specified conditions occurs.

Navigate to **Administration** > **Events** > **Subscriptions**.

[![Event Subscriptions](/assets/images/2017/09/vRA-event-subs.png)]({{site.url}}/assets/images/2017/09/vRA-event-subs.png)

Select **New** and from the list of Event Topics select ‘_Machine Provisioning_’. Click **Next**

[![New Subscriptions](/assets/images/2017/09/vRA-New-subscription-1.png)]({{site.url}}/assets/images/2017/09/vRA-New-subscription-1.png)

Set up the conditions as below

  * Data > Lifecycle State > State Name, Equals, VMPSMasterWorkflow32.MachineProvisioned
  * Data > Lifecycle State > State Phase, Equals, Post
  * Data > Machine, Machine Type, Equals Virtual Machine

[![Event Conditions](/assets/images/2017/09/vRA-new-subscription-conditions.png)]({{site.url}}/assets/images/2017/09/vRA-new-subscription-conditions.png)

Under the Workflow Tab, navigate through the vRO menu tree to ‘_EBS – Machine Provisioned – Chef_’. Click Next

[![Workflow Tree](/assets/images/2017/09/vRA-Event-subscription-WF-Tree.png)]({{site.url}}/assets/images/2017/09/vRA-Event-subscription-WF-Tree.png)


On the final tab, you can choose if the event will be blocking or not. A blocking event will run before any non-blocking event. If there are multiple blocking events, they will run in the priority order set.

[![Subscription Details](/assets/images/2017/09/vRA-Event-Subscription-Details.png)]({{site.url}}/assets/images/2017/09/vRA-Event-Subscription-Details.png)

The choice of using blocking here depends on your requirements for other post provision workflows.

After clicking **Finish** make sure to publish the new event subscription.

#### Starting a Workflow

Under the Catalog Tab, request the blueprint that you added the Chef property group too. If you select the VM and select Properties, you can view and edit the values for the properties in the property group. Additional configuration can prevent these values from being changed.

[![Request Properties](/assets/images/2017/09/vRA-Request-properties.png)]({{site.url}}/assets/images/2017/09/vRA-Request-properties.png)

After submitting the workflow, the Chef client will install on the new VM and perform normal bootstrap operations.

With some more work, you can populate these items from user input selections. For example, the value of ‘_runlist_’ can be set based on the application a user would like installed.

#### Summary

The Chef plugin has provided some easy options to get started with deploying the Chef client to provisioned VMs using vRA. Consider the properties as a getting started point, to get real value think of ways to expand how the age is deployed.

For example, the Chef server might be selected based on the vRA Business Group a User is a part of. Or the Chef environment could be selected based on the cluster a VM is deployed to.