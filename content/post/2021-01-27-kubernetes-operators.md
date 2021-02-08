---
author: Brett Johnson
postseries: operators
categories:
- Kubernetes
- Operators
date: "2021-01-27"
image: /assets/images/control-loop.svg
tags:
- Kubernetes
- Operators
title: Introduction to Kubernetes Operators
versions:
- software: Kubernetes
  versions:
  - v1.17.x
  - v1.18.x
  - v1.19.x
draft: false
---

This article is the first in a new series where I'll attempt to explain Kubernetes Operators and how to build them in a concise and easy to digest manner.

## What is an operator?

An operator is one or more custom resources and a control loop process which runs inside a pod on a Kubernetes cluster. 

## What does an operator do?

An operator manages a Kubernetes native application's lifecycle by extending the Kubernetes API using custom resources (CR's) specific to the application. A Kubernetes application is an application written specifically to run on Kubernetes and leverages the Kubernetes API.

Operators contain application-specific logic allowing automation of operational tasks required to deploy, update and remove an application. These operational tasks are manually performed by operations teams when no operator exists. 

Operators are targeted towards stateful services as they are more likely to require the execution of additional tasks. A stateless service's lifecycle can be managed by out of the box Kubernetes components as they generally do not require post-deployment configuration.

## Stateful vs Stateless services Operational tasks

Stateless services are designed to be ephemeral, deploying and destroying any number of identical pods as required. Pods provide a web service process may not even know about the existence of other pods running the same service; adding and removing these pods can be done by updating the number of replicas in a deployment.

Stateful services are not ephemeral and often require additional configuration tasks to deploy and manage. Database services are an example of a stateful service, where each pod in the database cluster will have a slightly different configuration to the rest. 

Scaling the number of pods providing our database service requires updating the service to add or remove new members. This kind of task is where operators can leverage application logic to automate the process.

## Custom Resources

Custom resources extend the Kubernetes API by adding a new end point to the collection of API objects.

A custom resources specification is defined in an object called a Custom Resource Definition (CRD). The CRD provides the Kubernetes API with metadata, data structure and validation information required to implement a new resource end point.

An operator applies at least one custom resource during deployment.

## The Control Loop

Operators run a control loop which is an infinite loop that cycles through three steps; 'Observe', 'Diff' and 'Act'.

![Control Loop](/assets/images/control-loop.svg)

**Observe**

The operator queries the Kube API to receive resources that it's configured to watch.

**Diff**
Resources received in the observation step are checked to see if they have changed and if current and desired states match. 
Most Kubernetes objects have a `.spec` field representing the desired state and a `.status` field representing the current state.

**Act**
The act step (AKA reconciliation function) attempts to align the current state of resources to the desired state by applying application logic.

Application logic is added to the reconciliation function and tells the operator what to do when a change is detected.

## Deployment Functionality Example

Let us look at how an operator might manage the lifecycle of a deployment from a functional level. 

Our deployment operator extends the Kubernetes resource API by providing the custom resource `deployment.apps`. 

We want to observe specific behaviours from our operator in response to changes to the custom resources it provides. Application logic needs to be added to the reconciliation function, telling the operator what to do. The table below provides several control loop examples, and the act column represents the application logic.

| Observe | Diff | Act |
| --- | --- | --- |
| Deployment | No replicaSet owned by this deployment in same namespace | Create repliaSet |
| Owned replicaSet | Owned replicaSet `.spec.replicas` does not match deployment `.spec.replicas` | Edit owned replicaSet `.spec.replicas` |
| Deployment | No change | Update status from owned replicaSet

## Summary

Operators help reduce the amount of manual operational tasks required to manage a Kubernetes Native applications lifecycle. Extending the Kubernetes API and running a control loop are a key part of operators.

In the next post I'll be covering the Operator Framework and Operator-SDK.