---
author: Brett Johnson
categories:
- Automation
date: "2017-09-04T20:30:45Z"
tags:
- API
- Automation
- Cloud
- infrastructure
title: 'Automation: Understanding the Role of your tool'
url: /BrettsITBlog/2017/09/automation-understanding-role-tool/
---

One of the challenges many of us come across when taking a more automated approach to infrastructure is the sheer number of tools available. This is especially true when you come from a one tool for multiple roles environment.

Every week, there appears to be another tool for you to choose from. Finding where to start is quite daunting, and the opinions of others leads to much second guessing. One of the lessons that would have helped me earlier on, is understanding what the role of a specific tool is and how it fits into the overall process.

Automation isn’t just about one task, it’s about linking many tasks together to achieve an outcome.

**Provisioners**

** **The role of a Provisioner to deploy and dispose of systems. The system could be anything from a VM to a VPC. The provisioner will need to be able to talk to other services such as; Hypervisors or Cloud Providers to make the required requests. Some provisioners will maintain an inventory of systems which they have deployed. Additionally, some will integrate with other tools such as configuration managers to extend their functionality.

**Configuration Managers**

** **Configuration managers are used to ensure the configuration of a system meets a described state. They can be used through the systems lifecycle and provide the ability to make incremental changes to that system. The general behavior is to only make a change when there is a variation of a specified setting, otherwise the operation is only read.

When assessing configuration managers, it’s important to understand the type of systems you will be managing. Some are stronger with servers, but weaker with networking devices.

**Compliance Managers**

Compliance managers are similar to Configuration Managers, but they aren’t intended to bring a system back into a desired state. Instead they would report on compliance against one or more baselines. The response to these could be contact the configuration manager and remediate or just alert.

The key difference is, a configuration manager is intended to resolve configuration drift at the time of detection, this might not be ideal is some cases.

**Orchestrators**

An orchestrator enabled multiple tasks to be stringed together into a workflow. Providing a platform to write custom code, enhancing logic and communication to multiple systems.

For example, to provision a System, the orchestrator will start the deployment workflow. After that workflow, has completed another workflow will talk to a configuration manager to start the application installation process.

**Management Platforms**

Management platforms can have a variety of uses, some provide a Web UI to submit workflows, others could provide the management functionality for compliance manager. Using Ansible as an example, by itself there’s no central management and scaling becomes difficult. The addition of Ansible Tower provides a central point of management, providing services such as HA and API.

When working with an Orchestrator, it’s much easier utilize a service with an API when it’s behind a management platform, then sending commands to a standalone service.