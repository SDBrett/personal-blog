---
author: Brett Johnson
categories:
- NSX
date: "2016-10-21T09:30:46Z"
id: 374
tags:
- Vmware
title: NSX Network and Security Functions and Services
url: /BrettsITBlog/2016/10/nsx-network-and-security-functions-and-services/
---

**Switching:** NSX switching resides on the data plane and utilises VMware vDS. Logical Switches are port groups on a vDS that are used for VXLAN traffic. Distributed port groups can also be used, but only for VLAN traffic.

**Routing:** Distributed routing, enabling routing to take place in kernel, without the need for traffic to enter the physical network. Dynamic routing is supported with OSPF, BGP and ISIS. Active / Active routing failover with physical routing.

**Distributed Firewall:** Firewalling takes place between the VMs vnic and the logical switch. Enabling ingress and egress scanning even on the same broadcast domain. Up to 20gps throughput per host. The DFW performs L2 – L4 firewalling, L7 firewall requires a 3<sup>rd</sup> party partner integration. NSX also provides North / South firewalling with the deployment of NSX Edge.

**Load Balancing:** L4 – L7 load balancing with SSL offloading. The load balancer has two operations modes; One Arm (Proxy Mode) and Inline (Transparent).

**NSX Gateway:** The NSX gateway (NSX Edge) , functions as a connection point between logical networks. An example would be an Edge gateway connects routes to a L2 bridge, this allows the VXLAN network to communication with the physical VLAN network. There are a number of services provided at the Edge, such as VPN.

**VPN:** The NSX Edge offers three VPN connection types, each with their own purpose and limitations.

SSL VPN-Plus: For remote users to connect into the work network. Web portal or installable VPN client

IPSEC VPN: Site to Site VPN connection. End point does not have to be an NSX Edge

L2 VPN: For extending L2 between sites. Traffic is sent over SSL using TCP443. Both end points need to be NSX Edge. Users a client / server relationship.

**API:** NSX management exposes API';s to allow programmatic configuration of the network and an integration point for 3rd party partners. This integration allows NSX to be consumed through self-service portals.

**Operations:** Central CLI, Traceflow, SPAN and IPFIX for monitoring and troubleshooting.

**Dynamic Security Policy:** NSX Service composer enables the use of dynamic security groups. Objects such as vCenter folders or security tags can be placed in a group, a security policy is then linked on that group.

**Cloud Management:** Integration with vRA and Openstack, amongst others.

**3<sup>rd</sup> Party Integration:** 3<sup>rd</sup> party partners can integration to the management and control plane, further extending the functionality of NSX. Adding services such and IDS and IPS, AV and increased visibility

**Cross vCenter networking and Security:** Extend networking and security vCenter and DC boundaries. Workloads can be migrated between data centres without the need to change L2 domains.

**Log management:** Log Insight for NSX integration. Improved logging, analysis and troubleshooting.