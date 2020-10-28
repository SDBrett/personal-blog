---
author: Brett Johnson
categories:
- VCAP7-CMA-Design
date: "2017-11-22T00:00:00Z"
description: Gather and analyze use cases
tags:
- VMware
- vRA
title: VCAP7-CMA Objective 1.2
---


When first reading the title of this objective it can seem a bit hard to view how it will be on the exam. After input from others, it appears that the key is analyze a given use case.

#### A Brief on Use Cases

The purpose of a use case is to tell a story based on the business goals. The use case will demonstrate a problem and what the business wants to do to solve it.

Sometimes, a use case will include key constraints. These are constraints which impact the solution options. Usually a driver for specific product choice over other options.

Client success stories are great places to view examples of use cases. Vendors use them to show where their products excel. Or, how their product is different from the compitition.

#### Mega Awesome Transit Use Case

For this section of the study guide, we will pull apart the provided use case and see what we can learn.

> Mega-Awesome Transit have identified a need to increase the reliability and reduce the complexity when updating their dispatching system during update periods.

The business problem stands out like a sore thumb. Reliability and complexity reduction are indicators of a need for cost reduction. Reliability can indicate availability requirements. Our design will need to simplify existing processes, while demonstrating an improvement to reliability.

> Currently updates to the dispatch system cause interruptions for up to 12 hours.

As this application is not one we have control over, this is difficult to directly address. Instead what are indirect measures our design can help? Through the design we should detail how we enable the business to resolve these issues. Improvements to the development environment is an improvement to the development output.

> Even though the networking, storage and compute infrastructure are highly virtualized, development and testing is slow due to the manual build process. As a result, only one development environment is created per update cycle.

Through the use of virtualization, we can assume there are options for on-demand environment builds. If this is validated, we will have options for deployment methods. Which link back to the previous subject paragraph. The use of "update cycle" is an sign of a fixed time to deliver. The less time waiting translates to more time coding and testing.

Hand on and manual processes are prone to errors. We could template a development environment builds. Removing the manual steps and decreasing deployment times, reduces risk and complexity.

> Mega-Awesome Transit would like to improve their deployment methods of the development environment to improve their ability to update their application and then look at new update strategies.

This is an important paragraph to read in full. The key wording is only the last 6 characters, the rest reads like a recap. Devil is in the details of a use case.

Ask yourself this. What products does VMware have which help code deployment? Codestream would be the answer. Even if not utilized on Day 1, our design needs to allow for implementation of a pipeline.

Our design needs to ensure required connectivity is present to push code to production. When deploying to brownfield sites, network air gaps are something to be aware of.


> Execs have heard that implementation of automated processes could allow developers to deploy their own environments. They expect providing developers control of development environments to yield efficiency gains. Resource and cost governance questions have been raised and will need to be addressed.

There is a lot of information packed into this paragraph.

We could make the assumption that there is no requirement for multi-tenancy. This is due to the lack of phrases indicating multiple companies or BU seperation.

We can improve efficency through the removal of manual steps. The context of the problem has refered to the development environment as a single entity.

It would stand to reason, efficency could be gained through the use of a composite blueprint. Which includes compute, on-demand networking and software components. An assumption could be made that this solution would also benefit from IPAM integration if available.

Generally there are governance questions cannot be answered at face value. We need to gather more information during discovery. Lets address options for resource and cost governance.

- Resource governance
    - Deployment limitsper user or blueprint
    - Limitation to size of deployed VMs
    - Approval policies
    - Reservations sizing
    - Reservation Policies


- Cost Governance 
    - Implementation of vRB 
        - Read what it can and cannot manage costs of
    - Cost display on form submittion
    - Cloud provider reservations
    - Approval policies
    - Business Manager notifications
    - vRB roles

#### Summary

Use cases provide information as to the what the business goals are and the key drivers behind those goals. Little details can often be missed on first reading. Before answering questions, be confident you know the full details. A single sentence can change an entire use case.

Think about what problems products within the exam scope solve and their limitations. An example is vRB does not integrate with Hyper-V. So it would an incorrect answer to a question on chargebacks with a Hyper-V endpoint.

<a class="item" href="/VCAP7-CMA-Design">Back to study guide</a> 