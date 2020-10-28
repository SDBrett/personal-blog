---
academia_post_display_home:
- ""
author: Brett Johnson
categories:
- Reviews
date: "2016-11-16T12:33:02Z"
id: 490
tags:
- Operations Management
- Turbonomic
title: 'Turbonomic Intro: Desired State and Virtual Market Place'
url: /BrettsITBlog/2016/11/turbonomic-desired-state/
---
Turbonomic is an operations management platform, built on a rich set of APIs. It has the capability to connect to many different systems and gather  extensive datasets.

This entry will cover the fundamentals of Desired State and Virtual Market Place. These two components are the basis for Turbonomics recommendations.

### Platform Support

A Target is a management system; such as vCenter. By connecting to a Target, Turbonomic can gather data and perform actions. Targets get grouped into the following catagories:

  * Hypervisors
  * Cloud Managers
  * Application Servers
  * Database Servers
  * Load Balancers
  * Storage Managers
  * Fabric Managers
  * Turbonomic

For the sake of providing up to date information, I';m not going to list all the supported platforms. To get this information, visit the Operations Manager User Guide.

### Virtual Market Place

Turbonomic refers to its capacity costing as a &#8216;Virtual Market Place';. By using a supply and demand model, resource cost can be calculated. The cost of a resource is an arbitrary number, not a dollar figure.

Lets look at this from a VM and Hypervisor perceptive. Each host has a finite number of resources. x GB of RAM, y total IOPs etc. When workloads are idle or light, demand is low, therefore cost is low. As workloads ramp up, such as people beginning their day in the office. Demand increases, raising the cost.

The costing based on supply and demand is the Economic Scheduling Engine.

### Supply Chain

Breaking down the costing model further, we look at the supply chain. A workload doesn';t just consume resources from one supplier. Basing a costing model on this would not produce accurate results.

The supply chain provides end to end visibility of resource consumption. Adding up the cost for resources in the chain provides total cost.

This is the Supply Chain view from the Turbonomic web page for a VM.

[![Supply Chain](/assets/images/2016/11/Turbonomic-Supply-Chain.png)]({{site.url}}/assets/images/2016/11/Turbonomic-Supply-Chain.png)

The above is a high level, more metrics go into the economic model. This is the not the post to go deep into the model, but that will be coming.

### Desired State

Each workload has different performance requirements. These are set by the business during design or required by the application to function. Turbonmoic cannot know our workload falls over if storage latency is above 2ms. We need to tell it this.

We need to configure policies which state the acceptable limits. This policy could apply to a; VM, Datastore, SQL application etc. Sticking with the VM example. We could say that storage latency on some of its virtual disks cannot exceed 2ms.

Configure policies as needed, Turbonomic will work out of the box. Too many policies introduce complexity and may reduce effectiveness.

By ingesting resource data and understanding policies (if configured) Turbonomic learns the &#8216;Desired State';. The desired state is the balance between efficient resource use and performance. Underutilization or resources is wasteful. But performance is at peak. Over utilization has a negative impact on performance. There is a balance where performance meets QoS and utilization is efficent. This is the desired state.

[![Desired State](/assets/images/2016/11/Turbonomic-Desired-State.png)]({{site.url}}/assets/images/2016/11/Turbonomic-Desired-State.png)

Performance as mentioned above, could be CPU ready time, disk latency etc. These metrics have an acceptable range. By saying performance dips with increased resource use, doesn';t imply negative impact. vCPU to pCPU ratio might change from 1:1 to 1.2:1, causing CPU ready to change from 1% to 1.4%. While this is a performance impact, it may not be detremental to the workloads running.

### Desired State and Cost

When Turbonomic knows the resource costs, it is able to made recommendations. These recommendations are to acheive desired state.

When starting with Turbonomic, let it run and collect. Input policies if needed. Let the product ingest enough data to provide accurate recommendations.

### Summary

Take time to understand the Cost model and Desired State. This understanding will help you get the most out of the product.