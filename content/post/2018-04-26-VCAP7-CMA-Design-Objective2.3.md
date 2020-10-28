---
author: Brett Johnson
categories:
- VCAP7-CMA-Design
date: "2018-04-26T00:00:00Z"
description: Build Availability Requirements into the Logical Design
tags:
- VMware
- vRA
title: VCAP7-CMA Objective 2.3
---

Availability requirements are those which refer to service resiliency to service fault. This is seperate from server availability as services could be redundant across multiple servers. 

Different services handling failures differently. Some have automatic recovery while others might need manual intevention.

To complete this objective, you will need to have an understanding of the underlying services. And their failure scenario characteristics. For example, what's the difference between Synchronous and Asynchronous DB replication on the vRA appliance?

In this section, certificate information has been included as the infomation on the certs based on the number of nodes for a given service.

#### vRA Appliance Availability

In order to have redundant vRA appliances, you need to have a minumum of two. This requires a load balancer VIP to be setup in front of the appliances. DNS and Certificates need to be configured for this configuration.

Load balancer configuration is detailed in the public VMware documents for vRA.

**DNS**

| DNS | Target | Purpose |
|---|--------|---------|
vra.company.local | 192.168.1.50 | Load Balancer VIP
vra-srv01.company.local | 182.168.1.51 | vRA appliance
vra-srv02.company.local | 182.168.1.52 | vRA appliance

**Certificate**
* CommonName: vra.company.local
* SAN:
  * vra.company.local
  * vra-srv01.company.local 
  * vra-srv02.company.local

Within VAMI of an appliance the 'HostName' should be that of the load balancer VIP. In our case vra.company.local.
To enable HA of the iDP, go to the vRA interface and change the connector URL to vra.company.local. This will send identity requests to the vip.

Services that connect to vRA should be configured to use the load balancer and not go direct to an appliance. For example if adding the vRA management pack to vROPs, the endpoint DNS entry will be vra.company.local.

When adding a new appliance to the cluster, deploy a new appliance via the OVA and skip the setup wizard. Within VAMI on the new appliance, go to cluster and enter the required details for the existing cluster. After clicking join the new appliance will retreive required data from the cluster and the postgres database to the cluster.

By default database synchronization between vRA appliance is asynchronous. With this configuration, only the master can write new data and the other appliances are in read-only mode. If the master appliance is not available, system functionality is reduced. To recover either the master node must come back online or someone must change another node to master through VAMI.

If you have deployed 3 or more appliances, the database synchronization can be set to synchronous. All appliances in the cluster can write to the database. If a node is unavailable, functionality is not reduced. Using synchronous replication has some considerations; appliances will increase resource usage and latency between appliances needs to be sub 5ms.

| Availability Requirement | Design Choice |
|--------------------------|---------------|
Lab / Dev / No HA requirement | Single vRA Appliance
HA required, small interuption ok | 2 vRA appliances
HA required, avoid interuptions | 3 + vRA appliances (scale for load)<br> Assumes sub 5ms latency between appliances
HA required, site redundant | 2 vRA appliances(one per site), async replication <br> Assumes intersite links sub 15ms latency

#### IaaS Manager Service

The IaaS manager service can be installed on multiple servers. In vRA 7.2 this is an active / passive configuration. DNS, Load Balancer and Certificate configurations need to be made to implement.

**DNS**

| DNS | Target | Purpose |
|---|--------|---------|
iaas-man.company.local | 192.168.1.56 | IaaS Web Load Balancer VIP
iaas-man-srv01.company.local | 182.168.1.57 | IaaS Web Server
iaas-man-srv02.company.local | 182.168.1.58 | IaaS Web Server

**Certificate**
* CommonName: iaas-man.company.local 
* SAN:
  * iaas-web.company.local 
  * iaas-man-srv01.company.local
  * iaas-man-srv02.company.local

When multiple IaaS managers are deployed, only one can be active. On any non-active IaaS Manager servers, the VMware <SERVICE> service should be disabled. If a failover occurs, then you must manually start the service.

| Availability Requirement | Design Choice |
|--------------------------|---------------|
Lab / Dev / No HA requirement | Single IaaS Manager 
HA required | 2 IaaS Managers


#### IaaS Web Service

The IaaS web service can be installed on multiple servers. In vRA 7.2 this is an active / active configuration. DNS, Load Balancer and Certificate configurations need to be made to implement. 

**DNS**

| DNS | Target | Purpose |
|---|--------|---------|
iaas-web.company.local | 192.168.1.56 | IaaS Web Load Balancer VIP
iaas-web-srv01.company.local | 182.168.1.57 | IaaS Web Server
iaas-web-srv02.company.local| 182.168.1.58 | IaaS Web Server

**Certificate**
* CommonName: iaas-web.company.local 
* SAN:
  * iaas-web.company.local 
  * iaas-web-srv01.company.local
  * iaas-web-srv02.company.local

Availability is quite simple for the web service, deploying more web servers increases failures to tollerate. Failover is automatic due to the active / active nature of the service.

| Availability Requirement | Design Choice |
|--------------------------|---------------|
Lab / Dev / No HA requirement | Single Web Server 
HA required | Multiple Web Servers

##### Summary

This section touches on some of the availabity requirements which need to be incorporated into a logical design. Additional considerations should be given to consolidation of services verus distributed, with management overhead and fault domains being a critical factor.
