---
author: Brett Johnson
categories:
- Cumulus
date: "2017-05-29T21:31:53Z"
tags:
- Cumulus
- Networking
title: 'Cumulus Linux: Network changes with a CICD pipeline'
url: /BrettsITBlog/2017/05/cumulus-linux-network-changes-cicd-pipeline/
---

While at Interop ITX 2017, I met with [Eric Pulvino](https://twitter.com/EricPulvino) from Cumulus Networks and learnt a bit about where Cumulus was heading and what’s new. One of the topics that got my attention was using a CICD pipeline for end to end testing of network configuration changes.

Cumulus Linux in a networking operating system (NOS) from Cumulus Networks. At its heart, this is a Debian Jesse distribution with additional networking smarts.

I';m not a networking person, but infrastructure and automation are right up my ally. So, this news really tickled my fancy. This post will cover the high-level concepts of using a CICD pipeline for network change validation.

Cumulus Linux in a networking operating system (NOS) from Cumulus Networks.

##### Reliable Testing

The Cumulus Linux platform is able to run on any x86 system, without configurational changes. The same Cumulus package can run on switching hardware or as a virtual machine. This flexibility allows for test builds to accurately represent production.

Configuration Management (CM) platforms; such as Ansible allow configurations to be deployed and validated just like they would be for server platforms. By using a CM, you can ensure that the test environment matches the current production configuration. Or vice-versa, validated test configurations can be pushed into production.

The ability to replicate configurations between test and production in an immutable fashion drastically increases the reliability of change and verification. When testing we make many little changes to get something to work, often unneeded configuration artefacts are left over. Through a CM, the test environment can be rolled back to match production, allowing you to only apply the required settings.

##### Testing at Scale

As Cumulus can run on any x86 platform, a test lab does not require specific hardware. Combined with the light weight nature of the Cumulus platform, test labs do not require a heavy investment in hardware. The lab could be as small as a laptop, or maybe decommissioned servers.

If your environment is leveraging specific ASIC features, switching hardware may be required for the test lab.

As configuration changes are made, the CICD pipeline can automatically prepare the test lab, deploying the required VM’s, applying configurations and running through test cases. This can speed up testing and verification of a change and additionally provide excellent data to keep the change management board happy.

##### End to End Testing

In the previous section, I touched on the CICD pipeline system deploying VM’s for switches. That case doesn’t really touch on the real benefit of introducing CICD practices into network configuration.

Network changes affect more than just the network devices, the big impact is on applications and services. Why not leverage the backup system as well?

It is becoming more common for backup systems to have a programmatic interface for operations. With proper dependency mapping, such as a failover plan you can increase you testing to include end to end application validation.

For example, we have a three-tier application which also requires Active Directory for authentication and DNS. These dependencies have been mapped out for our application. Our dependency mapping shows that our application as dependent on our switches and a few other servers. Our validation process spins up the switches and applies configurations, followed by restoring our servers into the test environment.

Validation of our change not only covers the networking devices, but not in addition verifies that our application will continue to function. This level of testing, will of course increase the test environment resources and rely on proper dependency mapping.

Depending on the backup system the application VM restore could take some time. However, there are some systems which provide ways to make this process much quicker.

All this could be done in the time it takes to enjoy a warm brunch with a quality Melbourne coffee. We have the best coffee, just saying.

##### Starting Small

The concepts details in this post cover processes that may be new to you and cover somewhat of a utopian environment. So, let';s scale back to provide a starting point.

Configuration management tools perform tasks from the simple to the complex. If you';re just dipping your toes in the water, think if simple by high impact tasks. Are you confident that NTP and DNS settings on all switches are correct? That';s a good starting place to understand CM, a small change but a very important setting to ensure is correct.

From there, you can add more tasks or logic, building the scale and complexity. At the same time, you could start looking at a CICD platform such as Jenkins and building a workflow to validate changes.

###### Summary

In this post, I focused on Cumulus Linux as the networking platform. In reality, the topics covered are not limited to Cumulus but can be applied to many different platforms.

By using techniques more commonly associated with software development, you can increase the reliability of your changes and reduce risk and headaches for all.

Thanks to Eric Pulvino for providing a technical review 🙂