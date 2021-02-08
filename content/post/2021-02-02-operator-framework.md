---
author: Brett Johnson
postseries: operators
categories:
- Kubernetes
- Operators
date: "2021-02-02"
image: /assets/images/operator_logo_framework_color.svg
tags:
- Kubernetes
- Operators
title: Kubernetes Operator Framework
versions:
- software: Kubernetes
  versions:
  - v1.17.x
  - v1.18.x
  - v1.19.x
draft: false
---

Development and management of Kubernetes native applications is a difficult task with a steep learning curve. The operator framework aims to reduce the complexity by pooling shared expertise into a single project and standardize application packaging.

Consumers benefit from operators' automated operations, making it easier to keep their applications up to date and secure. 

## The 3 Pillars

Three pillars underpin the operator framework. The section below provides an overview of each pillar, and follow-up posts will cover each one in more detail.

![Pillar Relationships](/assets/images/operator-pillar-relationships.svg)

**Build**

The build pillar provides the Operator SDK and resources to simplify the process of building an operator. The SDK leverages the [controller runtime project](https://github.com/kubernetes-sigs/controller-runtime) to construct new operators. The SDK provides an abstraction layer on top of the controller runtime, helping developers write add operational logic.

The build process creates a container image using the defined operational logic and the operator framework container image. The container image does not include CRD's or Role Based Access Control (RBAC) policies and cannot run.

Kustomize processes the CRD, RBAC, and Manager manifests and generates useable manifests. The resulting manifests contain everything required to run the operator on a Kubernetes cluster.

**Manage**

The manage pillar focuses on managing an operators lifecycle using the Operator Lifecycle Manager (OLM). The operator SDK consumes the manifests created during the build stage and creates a package called a bundle.

One of the artifacts created by bundling this the Cluster Service Version (CSV) which contains metadata for searching, minimum Kubernetes versions, required permissions and dependencies on other operators.

The OLM is an operator that runs on the kubernetes cluster, extending the API to enable operators to be managed.

OLM resources are not available on a cluster by default, you added them before using OLM. The following link contains details on deploying OLM resources. [Install an Operator](https://operatorhub.io/how-to-install-an-operator).

**Share and Discover**

The share and discover pillar provides the public operator registry Operatorhub.io where vendor and community operators can be published and consumed. The registry uses the CSV and CRD files generated using OLM to publish an operator, all operators published to a registry are lifecycle managed.

You can deploy a private operator registry service for internal applications or testing before making the operator public.

## Compatibility Model

The compatibility level specifies the application management maturity of an operator. 

Below are the compatibility levels and a brief overview of what each entails, you can find more information [here](https://operatorframework.io/operator-capabilities/).

**Level 1: Basic Install**
The operator can deploy an Operand or off cluster resources and determine when they are in a ready state. The operator provides a custom resource and reconciles configuration changes.

**Level 2: Seamless Upgrades**
The operator can perform the required tasks to upgrade custom resources and operands.
Upgrading the operator is seamless either through understanding older versions or automatically upgrading versions.

**Level 3: Full Lifecycle**
The operator can back up and restore the operand, perform complex orchestration and enable operand scaling.

**Level 4: Deep Insights**
The operator provides monitoring and alerting for itself and the operand.

**Level 5: Auto Pilot**
The operator automatically scales, heals and tunes the operand.

## Summary

This post has been a quick introduction to the purpose and pillars of the operator framework. Developers and consumers can benefit from the level of standardization provided by the framework, and it's ever-growing community.