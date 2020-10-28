---
author: Brett Johnson
categories:
- VCAP7-CMA-Design
date: "2017-11-21T00:00:00Z"
description: Gather and analyze business requirements
tags:
- VMware
- vRA
title: VCAP7-CMA Objective 1.1
---

Holding workshops with business stakeholders is the primary method of gathering business requirements. The goal of requirement gathering is to understand what the business is trying to acheive. Requirement workshops may also provide the constraints which we must work within.

As a consultant conducting design workshops, it is important to have an agenda and list of goals before starting a workshop. When planning a workshop determine the desired outcomes. This will determine the appropriate stakeholders to invite.

#### Stakeholders

Stakeholders are those in the business with an interest in the project. Each stakeholder will have a set of requirements that they need the project to meet. Their function within the business helps determine their role in the project.

| Stakeholders | Roles |
| :---| --- |
|C-Levels&nbsp;   | &nbsp; Business Planning <br> &nbsp; Long term planning <br> &nbsp; Budget <br> &nbsp; Business policies|
| Line Managers &nbsp; <br> BU Managers &nbsp;| &nbsp; BU specific requirements <br> &nbsp; Operational requirements <br>   |
| Application Owners &nbsp; | &nbsp; Opertional Requirements <br> &nbsp; BAU requrements <br> &nbsp; Manage Application functions <br>   |
| IT Administrators &nbsp; | &nbsp; Technical SME <br> &nbsp; BAU <br> &nbsp; Help desk <br>    |


<br>
Stakeholder interviews vary between businesses and sometimes engagements within the same business. When creating a design to solve a problem, it can be helpful to reaffirm previous decisions.

During the workshops below, we will gather requirements our design must meet.

#### Functional and Non-Funtional Requirements

There are two different types of requirements, functional and non-functional. 
Functional requirements describe what the design must achieve and parameters around that. A functional requirement could be; speed up the deployment of development environments. Or, show environment cost.

Non-functional requirements describe how the design will accomplish the goals. These will become constraints later in the design process. Development environments can only be deployed on AWS. A developer can only have 2 Development environments. 

#### Mega Awesome Transit Requirement Workshops

We have started with a brief use case, this is the kind of detail that can be expected at the begining of an engagement. Before we can create a design we need to find out what requirements our design must meet.

##### C Level Workshops

C level workshops, we should expect to gather business level requirements. Such as business objectives and legal regulations. These workshops are generally technical in nature.

##### CEO Requirements

* Decrease time to deploy updates
* Improve reliability of update process
* Reduce downtime of application updates

##### CFO Requirements

* Provide most cost-effective deployment environment
* Provide reporting on costs
* Enable governance over cost and prevent runaway pricing

##### CTO Requirements

* Provide flexibility of deployment location
* Automated testing of code
* Testing cannot impact performance of production 
* As there is no dedicated dev infrastructure, cap resource usage
* The solution should also allow IT staff to test VM's to cloud services

##### CSO Requirements

* Access from non-corporate networks (including VPN) requires extra authentication

##### C Level Summary

The requirements from our C levels are high level, but help gives an sign of what they would like the outcome to be. We have a mix of functional and non-functional requirements.

##### BU and Line Manager Workshops

The BU and Line managers how the solution fits within the BU. It will need to be in line with current standards.

Personal note: I have found a lot of assumptions get made during these workshops. It pays to verify with operation teams and application owners. 

#### BU and Line Manager Workshops

* Developers can only consume 25% of on-site infrastructure
* No more than 2 development instances per developer, maximum of 10 all up.
* Development environments should only exist as long as needed
* Reporting on test outcomes
* Deployments are to be set in size and configuration
* Developers work remotely and don't keep standard business hours. Downtime of the solution must be kept to a minimum.
* IT staff test VMs cost must be controlled, with larger instances requiring approval

##### BU and Line Manager

Our BU and Line Managers have a strong focus on governance. They want to ensure what we put in place does not have a negative impact on other systems. Their requirements represent constraints we must work within.

#### Application Owner Workshops

During these workshops, we gather requirements specific to the application and how it's configured. As an external party, this could be an application we have never heard of. Or even a bespoke application. Application owners will help us understand how it's put together and deployed. 

Personal note: It's not uncommon for application owners to contradict BU and Line managers. Usually, because the original deployment information is no longer relevant.

##### Application Owner Requirements

* Other systems integrate with the application through REST API
* End to end testing reports
* Developers need to create clean environments to validate tests
* The application data is sensitive to the company, but not regulated 
* Development environment resources must be as close to live as possible
* Use existing backup infrastructure to get data for testing
* Prevent deployments communicating with each other

##### Application Owner Summary

The application owners have focused on the specifics of the applications needs. Their requirements will help ensure reliability and consistency of development environments. In a real scenario, application owners would offer greater detail into deployment methods.

#### IT Administrators

IT administrator workshops help determine requirements such as; maintenance, operational support and alerts.

##### IT Administrator Requirements

* Use only specific networks
* Central logging system used
* Handle VM placements to prevent impact of other services
* Prefer cloud deployments to reduce risk of production service impact
* Limit communications to networks outside of deployment
* Test VMs in the public cloud should not touch production workloads


##### IT Administrator Summary

IT Administrator provide requirements for operations. Their requirements will assist BAU manage the platform.

##### Summary

This objective is about finding out what criteria our design must meet to gain business approval. Different management levels will have their own priorities. 

<a class="item" href="/VCAP7-CMA-Design">Back to study guide</a> 