---
author: Brett Johnson
categories:
- NSX
date: "2016-10-21T09:46:29Z"
id: 376
tags:
- Vmware
title: NSX Use Cases
url: /BrettsITBlog/2016/10/nsx-use-cases/
---
Whenever evaluating a product / technology, the question of "What problem does this solve?&#8221; should always be asked. Many times. This is how we understand the value and justify the expense, which can be very substantial. With that in mind, lets jump into some NSX Use Cases.

##### Security
NSX has a lot of features and capabilities, but security is probably the biggest draw card, especially micro-segmentation. In a traditional network, it’s very difficult and cumbersome to segregate workloads on the same L2 domain. This difficulty often means once the edge is breached, moving laterally isn’t a difficult and alerts are triggered. The ability to separate workloads on the same L2 domain and thus apply policy is called _Micro-Segmentation_. It’s difficult to have a conversation with someone about SDN and not have micro-segmentation come up.

NSX provides security filtering between the vNIC on the VM and the virtual switch. Meaning that policy is applied regardless of the destination.

Policy can be applied based on a huge number of factors, from IP address to vSphere objects. Policy can be dynamically applied based on VM names, which is great if there’s a solid naming standard.

But what if the naming standard is changing? Apply tags to the VM’s in vCenter, create a dynamic group based on the tag and apply the policy to the group.

##### VDI 
In this scenario, we are running a VDI solution, VMware View. Lets have a look at some NSX features and what they can help with.

When a desktop pool is created, the desktops within that pool are likely to be on the same L2 domain. This generally means that desktops can communicate freely to each other, also means that malware can spread East / West. Using the DFW for Micro-segmentation, we can stop this. Desktops can only communicate with the servers needed for the users to perform their tasks.

Being on the same L2 domain can limit the size of a desktop pool, due to broadcast concerns. NSX provides ARP suppression, allowing the desktop pools to be larger and reducing the number of pools to be managed.

3<sup>rd</sup> party partner integrations allow agentless AV. When desktops are provisioned there is no need to ensure that their AV is able to contact the management server. The base image might have an out of date AV agent install, meaning that a boot storm requires a large number of AV updates to be pushed. Agentless AV at the vnic level means that this is no longer the case.

##### Automation 
By providing management via REST API, network changes can be programmatically made and automation suites can automatically provision networking requirements to meet the need.

Workloads deployed through self-service portals can provision the required network connections and security without manual intervention. This is something critical to service providers for lowering costs, decreasing provisioning times and maximising profits.

**Security Automation: **When you look at a security incident, a breach of policy is detected and there is a response. Often there is manual intervention. One of the things that really has my attention with NSX is the possibility to automatically respond to breach / incident without manual intervention, and it can go very granular.

If my application was built on the principal of micro-segmentation and each segment is a Docker container, the traffic flow of each container would be very predictable making anomalies easy to detect. By being able to firewall at a container level, NSX can detect suspect container and respond. The response could be as simple as an alert or it could contact Docker Swarm and have the container removed, maybe even place it into an isolated network for examination and make sure that Swarm does not terminate the container.