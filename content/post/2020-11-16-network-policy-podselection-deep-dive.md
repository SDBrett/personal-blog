---
author: Brett Johnson
categories:
- Kubernetes
date: "2020-11-16"
image: /assets/images/Kubernetes_logo.svg
tags:
- Kubernetes
title: Network Policy Deep Drive P1 - Default policies
versions:
- software: Kubernetes
  versions:
  - v1.19.x
draft: false
---

This is the first part of a series of posts which will look into different types of network policies and create conflicting rules to observe the outcome. Tests in this series of posts have been performed using [Minikube](https://minikube.sigs.k8s.io/) with the [Cilium](https://cilium.io/) CNI, using a different CNI may change the observed results.


## Demo environment

The Kubernetes environment used for this post will start with the following:
- 2 Namespaces
  - np-deepdive-one
  - np-deepdive-two
- 2 types of pods
  - Web to test ingress policies
  - BusyBox to perform the tests from
- A service for the web pod

Additional objects will be created through out the examples.

### Manifests

**Busybox pod**
{{< highlight yaml >}}
apiVersion: v1
kind: Pod
metadata:
  labels:
    app: busybox
  name: busybox
spec:
  containers:
  - image: busybox
    name: busybox
    command:
    - sleep
    - "3600"
{{< / highlight >}}

**Web pod**
{{< highlight yaml >}}

apiVersion: v1
kind: Pod
metadata:
  labels:
    app: web
  name: web
  namespace: np-deepdive-one
spec:
  containers:
  - image: nginx
    imagePullPolicy: Always
    name: web
    ports:
    - containerPort: 80
      protocol: TCP
    command: [nginx-debug, '-g', 'daemon off;']
{{< / highlight >}}

**Web service**
{{< highlight yaml >}}
apiVersion: v1
kind: Service
metadata:
  labels:
    app: web
  name: web
  namespace: np-deepdive-one
spec:
  ports:
  - port: 80
    protocol: TCP
    targetPort: 80
  selector:
    app: web
  type: ClusterIP
{{< / highlight >}}


## Testing traffic flows

Traffic flow will be testing by running the following commands

**Traffic within the same namespace:**
{{< highlight shell >}}
kubectl -n np-deepdive-one exec -it busybox -- sh # access the busybox pods shell
wget -qO- --timeout=2 http://10-0-0-231.np-deepdive-one.pod.cluster.local # Test using pod IP address
wget -qO- --timeout=2 http://web # Test using service

{{< / highlight >}}


**Traffic different namespace:**
{{< highlight shell >}}
kubectl -n np-deepdive-two exec -it busybox -- sh # access the busybox pods shell
wget -qO- --timeout=2 http://10-0-0-231.np-deepdive-one.pod.cluster.local # Test using pod IP address
wget -qO- --timeout=2 http://web.np-deepdive-one.cluster.local # Test using service
{{< / highlight >}}


We will know that traffic flow is allowed the web server responds with the default nginx page

{{< highlight html >}}
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
    body {
        width: 35em;
        margin: 0 auto;
        font-family: Tahoma, Verdana, Arial, sans-serif;
    }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
{{< / highlight >}}

## Default Deny and Allow Policies

A default policy selects all pods in the namespace by setting the value of `.spec.podSelector` to `{}`, the value `{}` specifies no criteria to filter anything from the list.

### Default allow all ingress

The below manifest uses `.spec.ingress[]: {}` to state that any traffic sources are ok.

{{< highlight yaml >}}
---
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-all-ingress
spec:
  podSelector: {}
  ingress:
  - {}
  policyTypes:
  - Ingress
{{< / highlight >}}

Applying this to our namespace won't have any noticeable affect as the default state is to allow all traffic.

All traffic tests pass with this network policy in place.

### Default deny all ingress

To below manifest omits the `.spec.ingress` field, stating that there are not valid traffic sources.

{{< highlight yaml >}}
---
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: deny-all-ingress
spec:
  podSelector: {}
  policyTypes:
  - Ingress
{{< / highlight >}}

All traffic tests fail with this network policy in place as the traffic is not permitted into the web VM.


### Default allow all egress

To below manifest uses uses `.spec.egress[]:{}` to state that all destinations are valid.

{{< highlight yaml >}}
---
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-all-egress
spec:
  podSelector: {}
  egress:
  - {}
  policyTypes:
  - Egress
{{< / highlight >}}

All traffic tests will pass with this network policy in place.


### Default deny all egress

To below manifest omits the `.spec.egress` field which means there is no valid destination for traffic to go.

{{< highlight yaml >}}
---
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-all-egress
spec:
  podSelector: {}
  policyTypes:
  - Egress
{{< / highlight >}}

All tests fail with this policy applied.

### Default rule what if scenarios

**What happens if I apply both a default allow and a default deny ingress rule to the same namespace?**

If have applied both the deny and allow ingress policies to the `np-deepdive-one` namespace.

{{< highlight shell >}}
kubectl -n np-deepdive-one get networkpolicies.networking.k8s.io 
NAME                   POD-SELECTOR   AGE
allow-all-ingress      <none>         9s
default-deny-ingress   <none>         17s
{{< / highlight >}}

All tests pass because network rules are additive and the allow rule selects all pods. The same behaviour can be expected when applying a default deny and allow egress rules.

**What happens if I apply both a default allow ingress and a default deny egress rule to the same namespace?**

If have applied both the deny and allow ingress policies to the `np-deepdive-one` namespace.

{{< highlight shell >}}
kubectl -n np-deepdive-one get networkpolicies.networking.k8s.io 
NAME                  POD-SELECTOR   AGE
allow-all-ingress     <none>         62s
default-deny-egress   <none>         6s
{{< / highlight >}}

Tests from the busybox on on np-deepdive-one namespace fail. The nginx pod does not record any connection attempt in it's debug log.
Tests from the busybox on on np-deepdive-two namespace pass because the egress deny rule only applies to pods in the np-deepdive-one namespace.


**What happens if I apply both a default deny ingress and a default allow egress rule to the same namespace?**

If have applied both the deny and allow ingress policies to the `np-deepdive-one` namespace.

{{< highlight shell >}}
kubectl -n np-deepdive-one get networkpolicies.networking.k8s.io 
NAME                  POD-SELECTOR   AGE
allow-all-ingress     <none>         62s
default-deny-egress   <none>         6s
{{< / highlight >}}

All tests fail as we are blocking all traffic into the web pod regardless of the source. The web pod show no connection activity as the traffic is dropped before reaching the pod.

## Summary

In this post we looked at network policies which select all pods in the namespace and apply to all traffic sources and destinations; these policies are referred to as 'default' policies.

In the next network policy deepdive we will look at policies with different scopes and observe the behaviour of new conflicting policy sets.