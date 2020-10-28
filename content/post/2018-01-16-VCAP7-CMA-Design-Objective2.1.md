---
author: Brett Johnson
categories:
- VCAP7-CMA-Design
date: "2018-01-16T00:00:00Z"
description: Map Business Requirements to the Logical Design
tags:
- VMware
- vRA
title: VCAP7-CMA Objective 2.1
---

With your conceptual design accepted, you now need to create a logical design. This will create a map between requirements and components / services.

The CMA exam covers products with many configuraiton options. To help with clarity, this page will provide more generic information on the objective. I am currently working on a mindmap to link requirements to components and their options.

##### Purpose of a logical design

The logical design is, to an extent high level. Providing details on required components and service connections. In addtion, dependencies and communication paths. 

Logical designs are great for showing components and connectivity between said components. When using a visual representation, the detail can be quite light on. But, easy to consume. A mix of diagrams and text makes for a sucessful logical design.

Logical designs include items such as; Tenant roles, Tenants, Endpoints and Appliance resources.

##### Building the Logical Design from Requirements

When building the logical design, you need create a map between the requirement and component.  The process of mapping requirements calls for understanding of what each feature acheives. Also understand limitations.

vRB provides pricing information, but it only on certain platforms. It cannot meet a requirement to provide price information on Hyper-V.

While the detail in the logical design is high-level, the production knowledge is not.

TODO: Add Finish mindmap

##### Example question

"Mega Awesome Transport, want to add a cost based approval policies to their existing VM deployment blueprints. Pricing displayed should be based on the selected endpoint.
What should they do?"

* Deploy vRB, and create a new approval policy?
* Deploy vRB, enable location selection on existing blueprints and crete new entitlements
* Deploy vRB, enable location selection on existing blueprints and create a new approval policy?
* Deploy vROPS, Deploy vRB, enable location selection on existing blueprints and create new entitlements

I've made this a bit obscure on purpose, because sometimes things are left to "what if they want to x" and you need to go for the 'most right'.

2 and 4 could be ruled out because they do not add an approval policy. This is required to meet our requirement of approval based on cost.

1 is close as it will provide a cost, but it does not explicitly enable selection of the endpoint.

3 Covers the requirements better than the others and it's what I would select.

So, how does this relate to our objective? Lets map it out.

vRB will provide the costing information for number of hypervisor and cloud provides (not all).

vROPs isn't required based on information provided.

As the name suggests, requirements for approval mean the approval policy feature should be included.

The location selection feature of composit blueprints allows the user to select where their VMs will be deployed. This meets the requirement of of 'selected endpoint'.

##### Summary

The logical design is a high level design document detailing the components and services required. Building a logical design requires knowing how a component and service can solve a requirement.

<a class="item" href="/VCAP7-CMA-Design">Back to study guide</a> 