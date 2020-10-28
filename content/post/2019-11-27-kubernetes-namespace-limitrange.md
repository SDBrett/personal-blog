---
author: Brett Johnson
categories:
- Kubernetes
date: "2019-11-27T00:00:00Z"
image: /assets/images/Kubernetes_logo.svg
tags:
- Kubernetes
title: Namespace LimitRange
versions:
- software: Kubernetes
  versions:
  - v1.15.x
  - v1.16.x
---

A LimitRange object is used by cluster administrators to set default request and limit values for containers within a namespace. And minimum and maximum values for containers and pods within a namespace. 

If you're unfamiliar with container requests and limits within Kubernetes; [click here]({{< ref "/post/2019-11-25-kubernetes-resource-limits" >}} "Resource Limits").

Configuration a containers requests and limits setting is a well documented best practice. Cluster administrators use LimitRange objects to ensure that containers within a namespace align to this best practice.

## Default Limits and Requests

A LimitRange object has 4 settings for configuring default requests and limits values. The value requirements are the same as those used when configuring limits and requests for a container.

- `spec.limits.default.cpu`
- `spec.limits.default.memory`
- `spec.limits.defaultRequest.cpu`
- `spec.limits.defaultRequest.memory`

#### Default Limits and Requests Example

The below manifest sets default memory and CPU values for a namespace.

{{< highlight yaml >}}
apiVersion: v1
kind: LimitRange
metadata:
  name: limit-range
spec:
  limits:
  - default:
      memory: 512Mi
      cpu: 1
    defaultRequest:
      memory: 256Mi
      cpu: 500m
    type: Container
{{< / highlight >}}


{{< highlight shell >}}
$ kubectl create ns ranges
$ kubectl apply -f limitrange.yaml --namespace ranges
$ kubectl describe ns ranges
Name:         ranges
Labels:       <none>
Annotations:  <none>
Status:       Active

No resource quota.

Resource Limits
 Type       Resource  Min  Max  Default Request  Default Limit  Max Limit/Request Ratio
 ----       --------  ---  ---  ---------------  -------------  -----------------------
 Container  cpu       -    -    500m             1              -
 Container  memory    -    -    256Mi            512Mi          -

{{< / highlight >}}

Deploy a pod without configuring any limits or requests.

{{< highlight yaml >}}
apiVersion: v1
kind: Pod
metadata:
  name: limit-range-pod
spec:
  containers:
  - name: limit-range-ctr-01
    image: nginx
{{< / highlight >}}

{{< highlight shell >}}
$ kubectl apply -f limit-range-pod.yaml --namespace ranges
$ kubectl describe pod limit-range-pod.yaml --namespace ranges
Container
Limits:
  cpu:     1
  memory:  512Mi
Requests:
  cpu:        500m
  memory:     256Mi
{{< / highlight >}}

The containers Limits and Requests are set using the default values.

Let's look at the behaviour if we set only the limit for CPU.

{{< highlight yaml >}}
apiVersion: v1
kind: Pod
metadata:
  name: limit-range-pod
spec:
  containers:
  - name: limit-range-ctr-01
    image: nginx
    resources:
      limits:
        cpu: 1500m
{{< / highlight >}}

{{< highlight shell >}}
$ kubectl delete pod limit-range-pod.yaml --namespace ranges
$ kubectl apply -f limit-range-pod.yaml --namespace ranges
$ kubectl describe pod limit-range-pod.yaml --namespace ranges

Limits:
  cpu:     1500m
  memory:  512Mi
Requests:
  cpu:        1500m
  memory:     256Mi
{{< / highlight >}}

The container limit configuration took priority over the default value. The same behaviour occurs when configuring a limit with no requests at the container level.

Configuration of default limits causes a different behaviour to occur when a container configuration contains a requests value and no limits value.

{{< highlight yaml >}}
apiVersion: v1
kind: Pod
metadata:
  name: limit-range-pod
spec:
  containers:
  - name: limit-range-ctr-01
    image: nginx
    resources:
      requests:
        cpu: 1500m
{{< / highlight >}}

{{< highlight shell >}}
$ kubectl delete po limit-range-pod --namespace ranges
$ kubectl apply -f pod-memory-requests.yml --namespace ranges
The Pod "limit-range-pod" is invalid: spec.containers[0].resources.requests: Invalid value: "1500m": must be less than or equal to cpu limit
{{< / highlight >}}

When 
## Set Min and Max Requests and Limits

Cluster administrators can configure a minimum and maximum range for CPU and Memory resources used by containers within a namespace using LimitRange objects. If a containers resource configuration is not within the specified range, it cannot be deployed in that namespace.

Default requests and limits values are configured to match the max value if the namespace does not have defaults set. The default values can be configured in the same LimitRange manifest as min / max or in a separate manifest.

#### Container Min / Max Range

This section looks at configuring Min / Max ranges for containers within a namespace.

**Create a limit range**

{{< highlight yaml >}}
apiVersion: v1
kind: LimitRange
metadata:
  name: limit-range
spec:
  limits:
  - default:
      memory: 512Mi
      cpu: 1
    defaultRequest:
      memory: 256Mi
      cpu: 500m
    type: Container
{{< / highlight >}}

{{< highlight shell >}}
$ kubectl apply -f min-max-02 --namespace ranges
$ kubectl describe ns ranges
kubectl describe ns ranges
Name:         ranges
Labels:       <none>
Annotations:  <none>
Status:       Active

No resource quota.

Resource Limits
 Type       Resource  Min    Max  Default Request  Default Limit  Max Limit/Request Ratio
 ----       --------  ---    ---  ---------------  -------------  -----------------------
 Container  cpu       250m   1    1                1              -
 Container  memory    256Mi  1Gi  1Gi              1Gi            -

$ kubectl get limits min-max-01 -o yaml --namespace ranges
#SNIP
spec:
  limits:
  - default:
      cpu: "1"
      memory: 1Gi
    defaultRequest:
      cpu: "1"
      memory: 1Gi
    max:
      cpu: "1"
      memory: 1Gi
    min:
      cpu: 250m
      memory: 256Mi
    type: Container

{{< / highlight >}}

**Deploying a pod into the namespace without resources configured.**

{{< highlight yaml >}}
apiVersion: v1
kind: Pod
metadata:
  name: limit-range-pod
spec:
  containers:
  - name: limit-range-ctr-01
    image: nginx
{{< / highlight >}}

{{< highlight shell >}}
$ kubectl apply -f .\k8s\pod-limit-range-01.yml --namespace ranges
$ kubectl describe pod pod-limit-range-01 --namespace ranges
 Containers:
  limit-range-ctr-01:
    Limits:
      cpu:     1
      memory:  1Gi
    Requests:
      cpu:        1
      memory:     1Gi
{{< / highlight >}}

**Deploy pod that exceeds min/max values**

{{< highlight yaml >}}
apiVersion: v1
kind: Pod
metadata:
  name: limit-range-pod-02
spec:
  containers:
  - name: limit-range-ctr-02
    image: nginx
    resources:
      limits:
        memory: 2Gi
{{< / highlight >}}

{{< highlight shell >}}
$ kubectl apply -f .\k8s\pod-limit-range-02.yml --namespace ranges
Error from server (Forbidden): error when creating ".\\k8s\\pod-limit-range02.yml": pods "limit-range-pod-02" is forbidden: maximum memory usage per Container is 1Gi, but limit is 2Gi
{{< / highlight >}}

#### Pod Min/Max limits

This section looks at configuring Min / Max ranges for pods within a namespace. A pod object has requests and limits values equal to the sum of containers requests and limits within the pod.

**Create and apply a LimitRange object**
{{< highlight yaml >}}
apiVersion: v1
kind: LimitRange
metadata:
  name: min-max-02
spec:
  limits:
  - max:
      memory: 2Gi
      cpu: 1
    min:
      memory: 512Mi
      cpu: .5
    type: Pod
{{< / highlight >}}

{{< highlight shell >}}
$ kubectl apply -f .\k8s\min-max-02.yml --namespace ranges
$ kubectl get limits min-max-02 --namespace ranges -o yaml
# SNIP
spec:
  limits:
  - max:
      cpu: "1"
      memory: 2Gi
    min:
      cpu: 500m
      memory: 512Mi
    type: Pod
$ kubectl describe ns ranges
Name:         ranges
Labels:       <none>
Annotations:  <none>
Status:       Active

No resource quota.

Resource Limits
 Type  Resource  Min    Max  Default Request  Default Limit  Max Limit/Request Ratio
 ----  --------  ---    ---  ---------------  -------------  -----------------------
 Pod   cpu       500m   1    -                -              -
 Pod   memory    512Mi  2Gi  -                -              -
{{< / highlight >}}

**Deploy single container pod**

{{< highlight yaml >}}
apiVersion: v1
kind: Pod
metadata:
  name: limit-range-pod-03
spec:
  containers:
  - name: limit-range-ctr-01
    image: nginx
    resources:
      limits:
        memory: 1Gi
{{< / highlight >}}

{{< highlight shell >}}
$ kubectl apply -f .\k8s\pod-limit-range-03.yml --namespace ranges
Error from server (Forbidden): error when creating ".\\k8s\\pod-limit-range-03.yml": pods "limit-range-pod-03" is forbidden: [minimum cpu usage per Pod is 500m.  No request is specified, maximum cpu usage per Pod is 1.  No limit is specified]
{{< / highlight >}}

Unlike a container limit range, a pod limit range does not create any default requests or limits. This error is because default and defaultRequest are not allowed when the type is Pod.

{{< highlight yaml >}}
apiVersion: v1
kind: LimitRange
metadata:
  name: cpu-limit-range
spec:
  limits:
  - default:
      cpu: 1
    defaultRequest:
      cpu: 500m
    type: Pod
{{< / highlight >}}

{{< highlight shell >}}
kubectl apply -f .\k8s\limitrange-cpu-02.yml --namespace ranges
The LimitRange "cpu-limit-range" is invalid:
* spec.limits[0].default: Forbidden: may not be specified when `type` is 'Pod'
* spec.limits[0].defaultRequest: Forbidden: may not be specified when `type` is 'Pod'
{{< / highlight >}}

Let's add the container LimitRange from before and see what happens.

{{< highlight shell >}}
$ kubectl apply -f .\k8s\min-max-01.yml --namespace ranges
$ kubectl describe ns ranges
Name:         ranges
Labels:       <none>
Annotations:  <none>
Status:       Active

No resource quota.

Resource Limits
 Type       Resource  Min    Max  Default Request  Default Limit  Max Limit/Request Ratio
 ----       --------  ---    ---  ---------------  -------------  -----------------------
 Container  cpu       250m   1    1                1              -
 Container  memory    256Mi  1Gi  1Gi              1Gi            -
 Pod        cpu       500m   1    -                -              -
 Pod        memory    512Mi  2Gi  -                -              -

$ kubectl apply -f .\k8s\pod-limit-range-03.yml --namespace ranges
pod/limit-range-pod-03 created
$ kubectl describe pod limit-range-pod-03 --namespace ranges
# SNIP
Containers:
  limit-range-ctr-01:
    Limits:
      cpu:     1
      memory:  1Gi
    Requests:
      cpu:        1
      memory:     1Gi
{{< / highlight >}}


***Exceeding pod limits***

The application of defaults at the container level with limits at the pod adds a layer of consideration. Default values impose limits on the minimum and the maximum number of containers in a pod.

{{< highlight yaml >}}
apiVersion: v1
kind: Pod
metadata:
  name: limit-range-pod-04
spec:
  containers:
  - name: limit-range-ctr-01
    image: nginx
    resources:
      limits:
        memory: 1Gi
  - name: limit-range-ctr-02
    image: nginx
    resources:
      limits:
        memory: 1Gi
  - name: limit-range-ctr-03
    image: nginx
    resources:
      limits:
        memory: 1Gi
{{< / highlight >}}

{{< highlight shell >}}
$ kubectl apply -f .\k8s\pod-limit-range-04.yml --namespace ranges
Error from server (Forbidden): error when creating ".\\k8s\\pod-limit-range-04.yml": pods "limit-range-pod-04" is forbidden: [maximum cpu usage per Pod is 1, but limit is 3, maximum memory usage per Pod is 2Gi, but limit is 3221225472]
{{< / highlight >}}

```
Resource Limits
 Type       Resource  Min    Max  Default Request  Default Limit  Max Limit/Request Ratio
 ----       --------  ---    ---  ---------------  -------------  -----------------------
 Container  cpu       250m   1    1                1              -
 Container  memory    256Mi  1Gi  1Gi              1Gi            -
 Pod        cpu       500m   1    -                -              -
 Pod        memory    512Mi  2Gi  -                -              -

```

The combination of container and pod LimitRanges has interesting effects.
- A pod can have a maximum of 2 containers if no CPU requests or limits configured
- A pod can have a maximum of 2 containers if no memory requests or limits configured
- A pod can have a maximum of 4 containers due to the Container CPU minimum and Pod CPU maximum 
- A pod must have a minimum of 2 containers with minimum requests and limits set to the minimum

## Summary

LimitRanges are quite simple at face value, but, combining container and pod, LimitRanges increases complexity.
