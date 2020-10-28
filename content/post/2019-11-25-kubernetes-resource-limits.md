---
author: Brett Johnson
categories:
- Kubernetes
date: "2019-11-25T00:00:00Z"
image: /assets/images/Kubernetes_logo.svg
tags:
- Kubernetes
title: Kubernetes, Container Resource Requests and Limits
versions:
- software: Kubernetes
  versions:
  - v1.15.x
  - v1.16.x
---

Container CPU and Memory requests and limits configuration guarantees a minimum amount of resources to a container and sets a maximum consumable amount.

## Resource Configuration Values

Memory is measured in bytes and expressed as an integer or using a fixed point integer. For example; `memory: 1` is 1 byte, `memory: 1Mi` is 1 mebibyte / megabyte, `memory: 1Gi` is 1 gibibyte / gigabyte. Memory is not a compressible resource, and there is no throttling. Kubernetes evicts a pod from a node If a node cannot provide sufficient memory,

CPU is measure in millicpus, which is 1/1000th of a CPU core and expressed with integers. For example; `cpu: "1"` is 1 CPU, `cpu: 2000m` is 2 CPUs, `cpu: "0.75"` is 0.75 of a cpu and equivalent to `cpu: 750m`. CPU is a compressible resource and can be throttled by Kubernetes during contention. CPU contention does not cause pod eviction; instead, the app is slowed down.

## Requests

Requests represent the number of resources guaranteed to a container. This setting should be the same as or slightly above the minimum amount required for the container to function. The requests value for a pod is the sum of all requests for each container configured within that pod. A pod cannot start on a node unless the node has enough available resources for pods request value.

Configuring a low value for requests improves the chances that a container gets scheduled.

## Limits

A limit sets the maximum quantity of a Memory or CPU that a container can consume.  

Per container limits are configured within the relevant manifest file. There is no upper limit to the amount of memory or CPU that a container can consume if limits are not specified. 

Cluster administrators can configure default limits on a namespace using a `LimitRange`. Containers inherit namespace default limits if there are none configured at the container level.

If a container has a CPU or Memory limit set, but no requests value, the request value will equal the limit.


## Memory Examples

#### Example multi-container pod:

{{< highlight yaml >}}
apiVersion: v1
kind: Pod
metadata:
  name: mem-resource-pod
spec:
  containers:
  - name: mem-resource-ctr-01
    image: nginx
    resources:
      requests:
        memory: 256Mi
  - name: mem-resource-ctr-02
    image: nginx
    resources:
      requests:
        memory: 256Mi
{{< / highlight >}}

A node must be able to provide at least 512Mi of RAM, or the namespace must have 512Mi available for the pod to start.

#### Insufficent Memory Available:

{{< highlight yaml >}}
apiVersion: v1
kind: Pod
metadata:
  name: mem-resource-pod
spec:
  containers:
  - name: mem-resource-ctr-01
    image: nginx
    resources:
      requests:
        memory: 512Gi
  - name: mem-resource-ctr-02
    image: nginx
    resources:
      requests:
        memory: 512Mi
{{< / highlight >}}

{{< highlight shell >}}
$ kubectl describe pod mem-resource-pod
#SNIP
Events:
  Type     Reason            Age        From               Message
  ----     ------            ----       ----               -------
  Warning  FailedScheduling  <unknown>  default-scheduler  0/6 nodes are available: 6 Insufficient memory.
{{< / highlight >}}

There is no attempt from the scheduler to start a single container within the pod because there is no node with sufficient memory for the entire pod.

#### Memory limit set, no request

{{< highlight yaml >}}
apiVersion: v1
kind: Pod
metadata:
  name: mem-resource-pod
spec:
  containers:
  - name: mem-resource-ctr-01
    image: nginx
    resources:
      limits:
        memory: 512Mi
{{< / highlight >}}

{{< highlight shell >}}
$ kubectl describe pod mem-resource-pod
#SNIP
Limits:
  memory:  512Mi
Requests:
  memory:     512Mi
{{< / highlight >}}

Requests value is 512Mi because a limit is set and no request value was configured.

#### CPU Examples

A pod cannot be scheduled on a node if the CPU requests value is higher then the nodes total physical cores. Just like with memory, a pods CPU requests value is the sum of all containers within the pod.

The nodes in my lab have 2 CPU cores each and the pod configuration below requests a 1 CPU in total.

#### Example multi-container pod:

{{< highlight yaml >}}
apiVersion: v1
kind: Pod
metadata:
  name: cpu-resource-pod
spec:
  containers:
  - name: cpu-resource-ctr-01
    image: nginx
    resources:
      requests:
        cpu: 500m
  - name: cpu-resource-ctr-02
    image: nginx
    resources:
      requests:
        cpu: 500m
{{< / highlight >}}

This pod has no issues starting in the lab because the nodes have sufficient physical CPU.

#### Insufficent CPU Available:

{{< highlight yaml >}}
apiVersion: v1
kind: Pod
metadata:
  name: cpu-resource-pod
spec:
  containers:
  - name: cpu-resource-ctr-01
    image: nginx
    resources:
      requests:
        cpu: 1500m
  - name: cpu-resource-ctr-02
    image: nginx
    resources:
      requests:
        cpu: 1500m
{{< / highlight >}}

{{< highlight shell >}}
$ kubectl describe pod cpu-resource-pod
#SNIP
Events:
  Type     Reason            Age        From               Message
  ----     ------            ----       ----               -------
  Warning  FailedScheduling  <unknown>  default-scheduler  0/6 nodes are available: 6 Insufficient cpu.
{{< / highlight >}}

This pod could not start because the combined CPU requests exceeded the physical CPU cores.

#### Memory limit set, no request

{{< highlight yaml >}}
apiVersion: v1
kind: Pod
metadata:
  name: cpu-resource-pod
spec:
  containers:
  - name: cpu-resource-ctr-01
    image: nginx
    resources:
      limits:
        cpu: 1500m

{{< / highlight >}}

{{< highlight shell >}}
$ kubectl describe pod cpu-resource-pod
#SNIP
Limits:
  cpu:  1500m
Requests:
  cpu:        1500m
{{< / highlight >}}

The CPU requests match the limits value has no requests value is configured in the manifest.

## Summary

Configuring limits and requests is a Kubernetes management best practice which improves resource allocation and scheduling. It is a good idea to configure monitoring to detect where limits and requests are not configured. 