---
author: Brett Johnson
categories:
- VCAP7-CMA-Design
date: "2017-11-29T00:00:00Z"
description: Differentiate requirements, risks, constraints and assumptions
tags:
- VMware
- vRA
title: VCAP7-CMA Objective 1.3
---

During design workshops, we gather not only requirements, but also constraints which limit our choices. After inital workshops, we should building our design. Initially during this process we will need to make a number of assumptions and uncover potential risks. Assumptions need to be validated and risks raised throughout the process.

As a personal note, I have found the difference between Functional and Non-Functional requirements a bit of a struggle. I would highly recommend talking to others to help your learning.

#### Requirements

Requirements set the goals for a project. Stating what the design must achieve to be considered successful and the business drivers.
There are two types or requirements; functional and non-functional.

#### Functional Requirements
A functional requirement defines what solution needs to do. The use cases and goals that the design must achieve.
* Improve efficiency
* Provide costs
* Deploy to public cloud
* Reduce deployment times

#### Non-Functional Requirements
Non-functional requirements define how the design must achieve a goal. Generally speaking if a requirement doesn't fit into a category below, it's going to be functional.
Typical categories for non-functional requirements are:
* Recoverability
* Availability
* Manageability
* Performance
* Security

Our solution needs to provide cost information. We have number of related requirements from the workshops.

* BU Managers can view cost reports
* Development environments should only exist as long as needed
* Approval for deployments over specified sizes
The security team require that only secure connections to cloud providers are used. This is telling us how the connection to a cloud provider should be, thus is non-functional.

The previous workshops stated that developers need notification before maintenance periods. This is an availability requirement; maintenance operations cannot start without user notification.

#### Constraints
Constraints are items which limit our choices as architects. These will usually originate as non-functional requirements. If a client would like to clone from backup images to a cloud provider, but their backup system only supports AWS. We are constrained to use AWS.

Non-functional requirements that provide a limitation on architectural choices become constraints

When working with hybrid cloud deployments, network security policies can lead to a number of constraints. If a configuration manager is to be used to install applications on a cloud instance, we could find ourselves constrained by connectivity options.

In our scenario, it would be likely to see the following constraints:

* Network connection options to cloud providers
* Limitations from backup product for tasks such as cloning and live mounting
* Supported authentication methods
* On premises hardware resources
* Integration with existing tooling

#### Assumptions
Anything that we could not verify is an assumption. Try to verify all assumptions before sign off. Assumptions create risks, if proven incorrect it can impact the entire design.
The requirement for a secure connection to cloud providers, would lead us to assume that the network is capable. We should start contacting the networking team to confirm. Until that happens, document the assumption and related risk. If our assumption is incorrect, we may not be able to meet the requirement.

Example assumptions:
* Network able to support encrypted connection to cloud provider
* Additional hardware requirements will be met
* Second factor authentication can integrate with proposed solution

#### Risks
A risk is anything that could prevent a requirement being met. For each assumption, we should have a documented risk. Some risks are technical and others are business related.

Some common risks are:
* Staff training levels
* Encrypted network connection to cloud provider not confirmed
* Existing hardware is highly utilized
* Single WAN connection to cloud provider presents single point of failure

#### Applying to Use Case

---- In Progress ----
I'm currently creating a document containing the requirements, constraints, assumptions and risks. When completed this will be uploaded to github.

<a class="item" href="/VCAP7-CMA-Design">Back to study guide</a> 