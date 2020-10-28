---
author: Brett Johnson
categories:
- Kubernetes
date: "2019-10-04T00:00:00Z"
image: /assets/images/Kubernetes_logo.svg
tags:
- Kubernetes
title: Kubernetes, BYO workflows for efficent administration
versions:
- software: Kubernetes
  versions:
  - v1.15.x
---

I've recently started learning Kubernetes, working through the CKA curriculum and performing various administrative tasks. During my study so far, I have found that there are a lot of manual steps involved when looking at an end to end process. Often the process required to achieve an outcome requires the same data to be entered in multiple places, for example, labels and selectors. 

A process which I have found to be overly manual and prone to misconfiguration is user management. Discussions with friends and the community to better understand the process taught me an essential lesson for managing Kubernetes. 

Kubernetes provides the capability to achieve an outcome, but administrators need to build the workflows to efficiently and effectively manage the platform.

Kubernetes is intended to be managed using configuration files and code, and this can be confronting for many people starting with the platform. It is not viable to look at platform management as a series of individual tasks. Administrators must build workflows that interact with Kubernetes and other systems to achieve an outcome in a business suitable manner.

A workflow to grant user access to a Kubernetes cluster might take inputs such as username and an array of predefined roles. The workflow would then perform the following tasks:

- Validate username uniqueness
- Create a CSR for the new client certificate which will be used by the user
- Create certificates with using the trust CA
- Update database to track user access and roles
- Distribute certificates to the user
- Provide the user a kubectl config file
- Lookup RoleBindings and ClusterRoleBinding configuration files relevant to the selected roles
- Update the role configuration files by adding the user to the users' Subject field
- Commit changes to VCS
    - This will trigger another workflow to apply the updates to Kubernetes

The above process as many steps and would be prone to error if performed manually. Also, consider the additional steps required for business-specific requirements. Building a workflow to automate those steps for us will yield efficency and consistency improvements.

When looking to learn Kubernetes, take the time and consider how you would use individual tasks as part of a larger process to manage the platform.