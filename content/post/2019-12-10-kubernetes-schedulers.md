---
author: Brett Johnson
categories:
- Kubernetes
date: "2019-12-10T00:00:00Z"
image: /assets/images/Kubernetes_logo.svg
tags:
- Kubernetes
title: Schedulers
versions:
- software: Kubernetes
  versions:
  - v1.15.x
  - v1.16.x
---

Pods are not assigned to any node when they are first created; it is the job of the scheduler to assign the pod to a node. A scheduler watches the apiServer for pods that have no value in the `spec.nodeName` field. The scheduler finds the most suitable node for a pod-based on the podSpec and node statistics. After finding a suitable node, the scheduler sends an event to the apiServer, assigning the pod to the node. 

The kubelet watches the apiServer for pods are assigned to their node but not running. The kubelet detects a pod assign to its node, obtains the podSpec and executes the containers using its container runtime.

The scheduler sends an error to the apiServer if it is unable to find a suitable node. 

No scheduling occurs if `spec.nodeName` has a value specified in a pod manifest. This behaviour occurs because the manifest has performed the assignment already.

No scheduling occurs for static pods; those where the manifest file is located on a node (`/etc/kubenetes/manifests` by default). The local kubelet process manages static pods; not the apiServer.

The default Kubernetes scheduler uses a two-step process to assign pods to nodes.
**Filtering:** Filter out nodes which are not suitable, nodes that remain are `feasible node`. Filtering criteria examples:
- Taints and tolerances
- Labels
- Available resources

**Scoring:** Feasible nodes are scored against scoring rules, the pod is assigned to the node with the highest score. 

## Configuring the default scheduler

The default scheduler has many arguments available to fine-tune its configuration; these options can are found [here][6]. Options are configured as flags pushed to the scheduler pod or in the service file.

## Multiple Schedulers

A Kubernetes cluster can have multiple schedulers; additional schedules are implemented in a similar fas

You need to specify which scheduler a Kubernetes is to use for a pod using the `spec.schedulerName` field. If this is omitted, the default scheduler is used.

I haven't included examples in this post because I don't think I have anything to examples in the links below.

[Kubernetes Tasks: Configure multiple schedulers][2]
[Banzaicloud: Writing custom Kubernetes schedulers][3]

## Viewing scheduler events

The [cluster troubleshooting guide][4] states that the default log file location is `/var/log/kube-scheduler.log`, which is partly true. The Kubernetes installation method used influences where log files are stored if they are stored at all.

You should always ensure that deployment scripts specify log methods and paths. You can find information about configuring master service logs [here][5].

[1]: https://kubernetes.io/docs/concepts/scheduling/kube-scheduler/
[2]: https://kubernetes.io/docs/tasks/administer-cluster/configure-multiple-schedulers/
[3]: https://banzaicloud.com/blog/k8s-custom-scheduler/
[4]: https://kubernetes.io/docs/tasks/debug-application-cluster/debug-cluster/
[5]: {{< ref "post/2019-11-21-kubernetes-master-service-logging" >}}
[6]: https://kubernetes.io/docs/reference/command-line-tools-reference/kube-scheduler/