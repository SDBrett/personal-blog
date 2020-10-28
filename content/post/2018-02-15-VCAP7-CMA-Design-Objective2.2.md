---
author: Brett Johnson
categories:
- VCAP7-CMA-Design
date: "2018-02-15T00:00:00Z"
description: Map Service Dependencies
tags:
- VMware
- vRA
title: VCAP7-CMA Objective 2.2
---

Mapping service dependencies highlights internal and external communication requirements as well as improves the ability to troubleshoot effectively.

In some cases, dependant services are explicitly stated. If this is not the case pre-deployment steps can provide this information.

#### Upstream and Downstream Dependencies

Applications are composed of a number of services. Many of these services are dependent on another service, which is dependent on another. Look at the typical 3 tier application; Web, Application and Database. The service dependencies (at a very high level) would look like Apache > MyApp > MySQL.

The terms upstream and downstream are relative to the item within the chain that you're looking at. For example, if your vRA deployment is to use AD for authentication, AD would be downstream from vRA. However for AD to function, DNS must be working. So our chain needs to have DNS downstream from AD.

Downstream services are consumed by upstream, the failure of a downstream service should cause a failure on upstream services. Failure of upstream services should not impact downstream. 

As our automation components are dependent on many services, building the chain might appear to have chicken and egg scenarios. There's also a chance that you're going to really overcook the problem and not see the forest through the trees.

A common issue is that misconfiguration of an upstream product is considered. Using AD authentication as an example; if DNS is misconfigured in vRA AD authentication will fail. This does not affect the AD service itself, only the consumption. 

If AD is configured incorrectly upstream services will have errors, downstream services (in this case DNS) will continue to function.

![vRAServiceDependencies](/assets/images/vRAServiceDependencies.svg)

#### Dependency Scope

Dependencies can be assessed based on product or by task to be completed. Product scoped dependencies are those which are required simply for the product to be deployed. The task scope are additional to the product and depend on what the client wants to do.

If the client wants to use vRA to deploy instances in AWS, there would be a number of dependencies upstream from vRA. An example could be the VPN link to AWS, it's outage would not stop vRA functioning, but prevent the task from being completed.

#### vRealize Infrastructure Navigator

If you have read the links on the bottom of the exam blueprint, you'll know that vRealize Infrastructure Navigator (VIN) is listed.

Once configured, VIN will discover and graphically display service dependencies. VIN monitors systems to discover what services are running and what they are talking to. If you have a lab, spin it up and have a look around.

#### Summary

Understanding service dependencies is crucial to designing a solution and troubleshooting. Going through this process allows for more complete documentation and change requests. Especially when it comes to network connectivity.