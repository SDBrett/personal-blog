---
author: Brett Johnson
categories:
- VCAP7-CMA-Design
date: "2018-01-17T00:00:00Z"
description: Determine Use Cases for Multi-Tenancy
tags:
- VMware
- vRA
title: VCAP7-CMA Objective 4.1
---

When designing a vRA solution, a common question is "Does this use case fit business groups or multi-tenancy?". The key to answering this question is to determining what level of isolation entities require.

Business groups provide some level of separation; resources and entitlements. But there are times where tenant level settings need to differ. The only way to achieve this is to have mulitple tenants.

Some tenant level settings are inherited from the default tenant until overwritten within the tenant.

##### Visualization of Tenants 

This image from [blogs.vmware.com](https://blogs.vmware.com/management/2015/02/managing-multi-tenant-cloud-vrealize-automation.html) should help provide a visual guide to difference between System level and Tenant level.

[![Chef Environment](/assets/images/vRealize-Automation-Multi-Tenant.png)]({{site.url}}/assets/images/vRealize-Automation-Multi-Tenant.png)

##### Tenant Characteristics

Let's start by looking at the characteristics of a tenant. If these items need to be isolated to an entity, then you have a use case for multi-tenancy.

* Identity Source(s)
* Notification Settings
    * Email servers
    * Notification subscriptions
* Login URL
* Branding
* Business Policies
    * Approval Policies
    * Entitlements

The above items are configured per tenant and not shared with other tenants.

Identity sources need to be setup per tenant, even if the same source is used on another tenant.

If the client requires multi-tenancy and needs separation of vRO, external vRO appliances are reqiured.

##### Infrastructure Resources

As the IaaS Administrator is a system role, it is likely there would not be an IaaS admin per tenant. Instead the IaaS Administrator would configure endpoints and create fabric groups for all tenants.

The tenant would either be presented the fabric groups, or only have access to pre-created reservations.

>Example of presenting fabric group to tenants:
Mega Awesome Transport wants to use vRA to host an internal private cloud. Other business units purchase dedicated hardware for their consumption. Hardware resources are not shared between business units.

>Your design must ensure that business units only see their resources. Resource control is to be done by each business unit.

In this scenario, each tenant would have Fabric Administrators and the IaaS Administrator would configure a fabric group, assigned the tenants Fabric Administrators. This would allow each business unit to allocate resources as they decide.

Example of presenting reservations to tenants:
>Mega Awesome Transport has decided that they can better utilize infrastructure investment. They would like create a self-service portal for test / dev deployments on existing infrastructure. 

>Your design must ensure that limit can be placed on resource usage by test / dev deployments. Ensuring that production workflows are not impacted.

In this scenario, the IaaS Administrator would also be the fabric administrator. The same fabric group cluster would be presented to multiple tenants. The IaaS Administrator (using Fabric Administrator rights) would then create reservations, assigning those to business groups.

##### Applying to Mega Awesome Transport

Based on the requirements gathered there is no need to create multiple tenants. The use case is to build development environments, without specification that it requires management isolation.

The users are internal to the company and primarily made up of developers. There is a requirement to allow others to use vRA for deployments, with controls on resource usage. These can be managed with business groups, entitlements and approvals.

##### Summary

When evaluating the use case for multi-tenancy, assess the items to be isolated and the reason why. I work with a single tenant until something doesn't fit.

<a class="item" href="/VCAP7-CMA-Design">Back to study guide</a> 