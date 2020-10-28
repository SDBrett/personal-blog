---
author: Brett Johnson
categories:
- Chef
- vRA
date: "2017-09-07T09:08:05Z"
tags:
- Chef
- VMware
- vRA
- vRO
title: 'CHEF: Integration with vRA, installing the vRO plugin'
url: /BrettsITBlog/2017/09/chef-integration-vra-installing-vro-plugin/
---

The Chef agent is installed on a VM after the VM has been deployed and completed the ‘Machine Building’ stage of deployment. This is achieved by creating an event subscription through vRAs Event Broker Service (EBS).

When an event triggers an EBS subscription, vRA communicates with vRO causing a workflow to run. This means the first step of configuring Chef integration with vRA is to configure the Chef vRO plugin.

This post assumes some familiarity with vRO, vRA and Chef. To perform these tasks, your Chef server should be configured and running.

Steps detailed have been performed on vRA 7.3 with the embedded vRO appliance.

#### Integration with vRO

The Chef vRO plugin provides a number of prebuilt workflows for interacting with a Chef server and the object types for Chef components. Installation and configuration of the plugin in quite straight forward and required to use Chef with vRA.

Download the plugin from the VMware Solution Exchange or this [link.](https://marketplace.vmware.com/vsx/solutions/chef-plugin-for-vrealize-orchestrator)

To install the plugin to vRO, you need to navigate to the vRO control center. If you’re using the embedded vRO appliance, the control center is disabled by default. Follow the steps detailed in this [link](https://docs.vmware.com/en/vRealize-Automation/7.3/com.vmware.vra.prepare.use.doc/GUID-727FBB27-C440-4C95-B6B5-2B86C9E7D4F6.html) to activate.

##### Import Certs

The first step is to import the certificate for your Chef server. Follow the steps below from the vRO Control Center.

Click on Certificates

[![vRO Control Center](/assets/images/2017/09/vRO-Contol-Center-Certs.png)]({{site.url}}/assets/images/2017/09/vRO-Contol-Center-Certs.png)

Enter the URL of your Chef server and click Import

[![Enter URL](/assets/images/2017/09/vRO-Contol-Center-Input-URL.png)]({{site.url}}/assets/images/2017/09/vRO-Contol-Center-Input-URL.png)

##### Installing the Plugin

From the main page of the Control Center scroll down the page and click on Manage Plugins

[![Plugins](/assets/images/2017/09/vRO-Contol-Center-Plugins.png)]({{site.url}}/assets/images/2017/09/vRO-Contol-Center-Plugins.png)

Upload the Plugin

[![Upload Plugin](/assets/images/2017/09/vRO-Contol-Center-Install-plugin.png)]({{site.url}}/assets/images/2017/09/vRO-Contol-Center-Install-plugin.png)

Accept the EULA and click Install

[![Accept EULA](/assets/images/2017/09/vRO-Contol-Center-Accept-EULA.png)]({{site.url}}/assets/images/2017/09/vRO-Contol-Center-Accept-EULA.png)

If you’re running a vRO cluster, you need to restart the vRO services to synchronize the change.

#### **Connecting the Chef Server as an Endpoint**

For vRO to communicate with the Chef server, you will need to configure an endpoint.

Launch the vRO client and go to Workflows > Library > Chef > Configuration

Select Add Chef Host

[![Add host](/assets/images/2017/09/vRO-Client-Add-Chef-Host.png)]({{site.url}}/assets/images/2017/09/vRO-Client-Add-Chef-Host.png)

Enter the required data and click Submit

[![Add host Data](/assets/images/2017/09/vRO-Client-Chef-Add-Host-Data.png)]({{site.url}}/assets/images/2017/09/vRO-Client-Chef-Add-Host-Data.png)

After the workflow completed, your Chef server has been added as an endpoint. If the work fails, validate your input and correct where required.

You can verify the connection to the Chef server using the workflow “Chef Server

[![Client status check](/assets/images/2017/09/vRO-Client-Check-Chef-Status.png)]({{site.url}}/assets/images/2017/09/vRO-Client-Check-Chef-Status.png)

The output of the workflow should be similar to the image below

[![Client status output](/assets/images/2017/09/vRO-Client-Check-Chef-Status-Output.png)]({{site.url}}/assets/images/2017/09/vRO-Client-Check-Chef-Status-Output.png)

#### Summary

This completes the vRO stage of integrating Chef with vRA. In the upcoming posts, we will look at the EBS and property groups to make everything work.