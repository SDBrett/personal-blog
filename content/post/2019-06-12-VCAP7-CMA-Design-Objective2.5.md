---
author: Brett Johnson
categories:
- VCAP7-CMA-Design
date: "2019-06-12T00:00:00Z"
description: Build Performance Requirements into the Logical Design
tags:
- VMware
- vRA
title: VCAP7-CMA Objective 2.5
---

Performance requirements play a significant role in the architecture and configuration of a vRA design. Understanding the capacity limits and functions of each component within the vRA solution will help you design a suitable architecture to meet the clients needs. It can be challenging to understand the expected load if you're working with a client who is new to automation; thankfully, it is quite easy to add another component as demand increases.

It is essential to understand the function of each component and the capacity limits as components can be scaled individually to meet a specific performance requirement. Increasing solution scale increases performance/capacity without changes to the underlying hardware platform, at the expense of increased management overhead.

Troubleshooting, in particular, becomes much more difficult as the solution architecture becomes more distributed; a central logging solution can help to reduce this difficulty.

#### Topology scaling

The simple deployment architecture that has all IaaS components on a single Windows server is not able to be scaled; therefore should not be considered for production deployments.

At a minimum a production vRA deployment should contain the following (Where DEM roles are on either IaaS manager or IaaS web servers):
- 2 x vRA Appliance
- 2 x IaaS Manager Server
- 2 x IaaS Web Server
- 2 x MSSQL Server

You can halve the number of servers if HA is not a requirement, though this is typically not considered suitable for production. In this case, Load balancers VIPs are not required. However, it can be challenging to add them later if you do not create CNAME DNS records for the vRA appliance and IaaS services. The CNAME records become the FQDN for a load balancer VIP when the solution needs to be scaled out and have multiple IaaS servers or vRA appliances.

For example before scaling the CNAME record `vra.company.local` would point to the FQDN of the vRA appliance `vra01.company.local` and the CNAME record would be used as if it were a VIP FQDN in a clustered appliance solution. 
When a new vRA appliance is required, you will perform these steps:

- Configure load balancer (VIP, Pools, Health checks etc.)
- Change DNS record `vra.company.local` to an A record that points to the new VIP IP address
- Update CA cert for vRA appliance
- Deploy new appliance

The above steps are only a high level and do not capture everything involved. A similar process is used for the IaaS Manager and Web servers.

#### Handling a large active directory

vRA authentication services are provided by an instance of vIDM. The vIDM instance is allocated a percentage of the memory allocated to a vRA appliance VM; each AD Object synchronized through the directory synchronization process increases the memory consumed by the vIDM service. Using synchronization filtering options reduces the number of objects synchronized and overall memory consumption. The amount of memory allocated to a vRA appliance will need to be increased if Active Directory is too large and filtering does not provide a sufficient memory reduction.

The directory setting `Base User DN` within the directory configuration page defines the LDAP search query scope. Users may experience long login times if the scope is too broad, however, setting the scope too narrow may exclude user accounts is some OU's preventing them from logging in. The correct setting for `Base User DN` depends on the AD structure and with the AD User objects are stored.


#### Data collection

vRA periodically runs a data collection process against configured endpoints, this process consumes a DEM workers execution process. The design must account for data collection from endpoints with a large inventory and/or data collection from many endpoints. Either scenario impacts the deployment or configuration of DEM components.

A decision will need to be made regarding advanced configuration settings; such as adjusting the maximum number of concurrent workflows, or, deploying more DEM components.

#### DEM proxy agents

DEM proxy agents are used to connect to endpoints for tasks such as Data Collection. Agents should be placed as close as possible to the endpoints that they are connected to. Placing an agent in the same datacenter/cluster as the vCenter that it collects data from significantly increases data collection performance. 

Proxy Agents should be placed as close to the edge as possible for cloud endpoints such as Azure or AWS.

#### Request forms

Fields on request forms can be linked to vRO actions for dynamic value population and validation. Dynamic values have an overhead when they first load and when their data needs to be refreshed. An API call is made to vRO, the action is run and then the results are returned to vRA to be displayed on the form. Running actions for dynamic request form fields increases the load on the vRO service and needs to be taken into account when assessing vRO scale. 

Form speed can be improved in a few ways:
- Check your code, make sure it's efficient
- If the action takes values from other field values
  - Handle `null` input first
  - Handle as much input validation upfront as possible
- If external information is required
  - Consider the use of resource elements to cache responses
  - Limit the number of REST calls
- Use tabs to lower the number of actions being run simultaneously