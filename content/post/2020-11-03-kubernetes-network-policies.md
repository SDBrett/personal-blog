---
author: Brett Johnson
categories:
- Kubernetes
date: "2020-11-02"
image: /assets/images/Kubernetes_logo.svg
tags:
- Kubernetes
title: Network Policies
versions:
- software: Kubernetes
  versions:
  - v1.19.x
draft: false
---

Kubernetes network policies allow platform consumers to restrict layer 3 and 4 network traffic. All ingress and egress traffic is permitted (default-allow) for namespaces without network policies in place. When at least one network policy exists, any traffic that does not match the specified rules is dropped (default-block).

It is essential to be mindful of the default allow state when providing a Kubernetes service to users as it could impact platform and service design decisions. 

I worked with a client where security was paramount, particularly preventing communication between different applications using the Kubernetes platform. The initial plan was to use a shared cluster and allocate one namespace to each application; however, we ended up implementing a dedicated Kubernetes cluster per application. Namespaces having a default allow traffic flow was the primary factor for changing to a cluster per application model.

The cluster CNI is responsible for the implementation of the network policy rules applied to namespaces. Some CNI's such as Flannel, do not support network policies at all.

## Pod Selection

The field`.spec.podSelector` specifies the scope of pods within the namespace to which the policy applied. The subfields `matchExpressions` and `matchLabel` specify the criteria for eligible pods.

## Policy Types

The types of policies specified by the `.spec.policyTypes` field, however. The field expects an array of strings where 'Ingress' and 'Egress' are the only legal values. This field is optional, but, there are some behaviours to be aware of if you choose to omit it.

- Ingress will be assumed
- Egress is set if egress rules exist.

The example below demonstrates the behaviour when no policy types or policies are specified.

**Manifest applied**

{{< highlight yaml >}}
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: no-policy-type
spec:
  podSelector:
    matchLabels:
      role: demo
{{< / highlight >}}

**Network policy created**

{{< highlight yaml >}}
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: no-policy-type
  namespace: default
spec:
  podSelector:
    matchLabels:
      role: demo
  policyTypes:
  - Ingress
{{< / highlight >}}

The `.spec.policyTypes` field is added to the new object with the value of ['Ingress'] despite no ingress rules specified.

Let us look at what happens with no policy types set and an egress rule added.

**Manifest applied**

{{< highlight yaml >}}
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: no-policy-type-no-egress
spec:
  podSelector:
    matchLabels:
      role: demo
  egress:
  - to:
    - ipBlock:
        cidr: 192.168.1.0/24
{{< / highlight >}}

**Network policy created**

{{< highlight yaml >}}
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: no-policy-type-no-egress
  namespace: default
spec:
  egress:
  - to:
    - ipBlock:
        cidr: 192.168.1.0/24
  podSelector:
    matchLabels:
      role: demo
  policyTypes:
  - Ingress
  - Egress
{{< / highlight >}}

This time we set an egress rule which added 'Egress' to `.spec.policyTypes` and just like the first example, 'Ingress' was added to `.spec.policyTypes` automatically.

## Ingress

Ingress rules specify the traffic is allowed to reach the pods in scope according to `.spec.podSelector`. Network Policies perform layer 3 and 4 filtering, which means we can filter based on port and protocol.

The `.spec.ingress[]` field contains an array of ingress objects. An ingress option is composed of two main parts; `.spec.ingress[].from[]` and `.spec.ingress[].ports[]`. The `from[]` object is used to specify permitted traffic sources. The `ports[]` specifies the allowed destination ports and protocols.

Putting the above together, pods which match criteria configured in `.spec.ingress[x].from[]` are allowed to send traffic to pods which match criteria configured in `.spec.podSelector` with ports and protocols on `.spec.ingress[x].ports[]`.

After an ingress policy is applied to a namespace, traffic which does not match the allowed rules is dropped.

### Deny all Ingress

An ingress rule which does not specify any `.spec.ingress[]` object will deny all incoming traffic to pods in the scope of `.spec.podSelector`. The example below denies all traffic to all pods in the namespace.

{{< highlight yaml >}}
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: deny-all-ingress
spec:
  podSelector: {}
  policyTypes:
  - Ingress
{{< / highlight >}}

### Allow all Ingress

An ingress rule with an empty `.spec.ingress[]` will allow all traffic to pods in the scope of `.spec.podSelector` from any source. The example below denies all traffic to all pods in the namespace.

{{< highlight yaml >}}
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-all-ingress
spec:
  podSelector: {}
  policyTypes:
  - Ingress
  ingress: 
  - {}
{{< / highlight >}}

## Egress

Egress rules specify allowed destinations for traffic sent by pods in the scope of `.spec.podSelector`. Just like egress rules, egress rules filter by port and protocol.

The `.spec.egress[]` field contains an array of egress objects. An egress option is composed of two main parts; `.spec.egress[].to[]` and `.spec.ingress[].ports[]`. The `from[]` object is used to specify permitted traffic sources. The `ports[]` specifies the allowed destination ports and protocols.

Putting the above together, pods which match criteria configured in `.spec.podSelector` are allowed to send traffic to destinations configured in `.spec.egress[x].to[]` using ports and protocols specified in `.spec.egress[x].ports[]`. 

After an egress policy is applied to a namespace, traffic which does not match the allowed rules is dropped.

### Deny all Egress

An egress rule which does not specify any `.spec.egress` object will deny all outgoing traffic. The example below denies all traffic from all pods in the namespace to any destination.

{{< highlight yaml >}}
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: denyall-egress
spec:
  podSelector: {}
  policyTypes:
  - Egress
{{< / highlight >}}

### Allow all Egress

An egress rule with an empty `.spec.egress` will allow all traffic from pods in scope of `.spec.podSelector`. The example below allows all traffic from all pods in the namespace to any destination.

{{< highlight yaml >}}
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

## Summary

Network policies are a great tool to ensure network traffic isolation for Kubernetes workloads. They can take a little bit to understand at the start, but, the learning curve is eased slightly by using the same pod selection methods we've seen for other tasks such as services and replicaSets.

One common hurdle I've experienced when working with clients is the application developers have never had to do network security before and take a lot of hand-holding to help them along the way. Challenges here echo to security and compliance teams wanting to ensure particular standards are maintained while not trusting the application teams abilities. There is no technical way to address these challenges, just persistence, right wording and patience.