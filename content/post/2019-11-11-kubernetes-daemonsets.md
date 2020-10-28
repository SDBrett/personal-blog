---
author: Brett Johnson
categories:
- Kubernetes
date: "2019-11-11T00:00:00Z"
image: /assets/images/Kubernetes_logo.svg
tags:
- Kubernetes
title: Kubernetes, Daemonsets
versions:
- software: Kubernetes
  versions:
  - v1.15.x
---

A DaemonSet is an object which ensures that there is a copy of a pod running on each node. This type of object is commonly used to deploy pods which provide a management service, such as log collection daemons or monitoring daemons to provide data to a solution such as Prometheus.

The scheduler places a copy of the pod specified in a DaemonSet on all eligible nodes within the cluster.

The manifest for a DaemonSet is very similar to that of a Deployment. There are a few significant differences to be aware of:
- The `spec.template.RestartPolicy` field must have a value of 'Always' or not be specified. If the field is not present in the manifest, the pod has a restart policy of 'Always'.
- There is no field for `spec.replicas` as DaemonSet pods run on all nodes.

The fields `spec.template.spec.nodeSelector` and `spec.template.spec.tolerations` can be used to control scheduling with labels and taints.

The below manifest is based on the example from the [Kubernetes Documentation](https://kubernetes.io/docs/concepts/workloads/controllers/daemonset/).

{{< highlight yaml >}}
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: fluentd-elasticsearch
  namespace: kube-system
  labels:
    k8s-app: fluentd-logging
spec:
  selector:
    matchLabels:
      name: fluentd-elasticsearch
  template:
    metadata:
      labels:
        name: fluentd-elasticsearch
    spec:
      containers:
      - name: fluentd-elasticsearch
        image: quay.io/fluentd_elasticsearch/fluentd:v2.5.2
        resources:
          limits:
            memory: 200Mi
          requests:
            cpu: 100m
            memory: 200Mi
        volumeMounts:
        - name: varlog
          mountPath: /var/log
        - name: varlibdockercontainers
          mountPath: /var/lib/docker/containers
          readOnly: true
      terminationGracePeriodSeconds: 30
      volumes:
      - name: varlog
        hostPath:
          path: /var/log
      - name: varlibdockercontainers
        hostPath:
          path: /var/lib/docker/containers
{{< / highlight >}}

The above manifest schedules a pod on each eligible node in the cluster.

{{< highlight sh >}}
$ kubectl apply -f daemonset.yaml 
$ kc get ds -n=kube-system
NAME                    DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR                 AGE
calico-node             6         6         6       6            6           beta.kubernetes.io/os=linux   25d
fluentd-elasticsearch   6         6         6       6            6           <none>                        3s
kube-proxy              6         6         6       6            6           beta.kubernetes.io/os=linux   25d

$ kubectl get pods -n=kube-system --selector name=fluentd-elasticsearch -o wide
NAME                          READY   STATUS    RESTARTS   AGE   IP              NODE     NOMINATED NODE   READINESS GATES
fluentd-elasticsearch-c5r58   1/1     Running   0          2s    10.10.165.235   k8s-03   <none>           <none>
fluentd-elasticsearch-k22jg   1/1     Running   0          2s    10.10.179.42    k8s-02   <none>           <none>
fluentd-elasticsearch-zh45g   1/1     Running   0          2s    10.10.61.224    k8s-01   <none> 
{{< / highlight >}}

The manifest did not include any taint tolerations; therefore, the scheduler did not place pods on masters.

{{< highlight sh >}}
$ kubectl describe nodes k8s-master-01 | grep Taints
Taints:             node-role.kubernetes.io/master:NoSchedule
{{< / highlight >}}

Updating the manifest field `spec.template.spec.tolerations` enables scheduling of pods on masters.

{{< highlight yaml >}}
spec:
  template:
    spec:
      tolerations:
      - key: node-role.kubernetes.io/master
        effect: NoSchedule
{{< / highlight >}}

{{< highlight sh >}}
$ kubectl get pods -n=kube-system --selector name=fluentd-elasticsearch -o wide
NAME READY   STATUS        RESTARTS   AGE     IP              NODE            NOMINATED NODE READINESS GATES
fluentd-elasticsearch-22f65   1/1     Running       0          7s      10.10.95.8      k8s-master-02   <none>           <none>
fluentd-elasticsearch-8db4n   1/1     Running       0          7s      10.10.151.130   k8s-master-01   <none>           <none>
fluentd-elasticsearch-c5r58   1/1     Running       0          8m43s   10.10.165.235   k8s-03          <none>           <none>
fluentd-elasticsearch-k22jg   0/1     Terminating   0          8m43s   10.10.179.42    k8s-02          <none>           <none>
fluentd-elasticsearch-wj7v5   1/1     Running       0          7s      10.10.183.130   k8s-master-03   <none>           <none>
fluentd-elasticsearch-zh45g   1/1     Running       0          8m43s   10.10.61.224    k8s-01
{{< / highlight >}}

There might be a use case to run a service provided by a daemon set to only a subset of nodes within the cluster. Node scheduling behaviour is the same as with Pod and Deployment configurations.

The manifest field `spec.template.spec` has been updated with a nodeSelector configuration and labels have been placed on two of the 3 nodes. 

{{< highlight yaml >}}
spec:
  template:
    spec:
      nodeSelector:
        logger: fluentd
{{< / highlight >}}

{{< highlight sh >}}
$ kc label node k8s-01 logger=fluentd
$ kc label node k8s-02 logger=fluentd
$ kc get ds -n=kube-system
NAME                    DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR                 AGE
calico-node             6         6         6       6            6           beta.kubernetes.io/os=linux   25d
fluentd-elasticsearch   2         2         2       2            2           logger=fluentd                25m
kube-proxy              6         6         6       6            6           beta.kubernetes.io/os=linux   25d
$ kc get pods -n=kube-system --selector name=fluentd-elasticsearch -o wide
NAME                          READY   STATUS    RESTARTS   AGE     IP             NODE     NOMINATED NODE   READINESS GATES
fluentd-elasticsearch-26crr   1/1     Running   0          3m17s   10.10.179.44   k8s-02   <none>           <none>
fluentd-elasticsearch-t9dc5   1/1     Running   0          3m4s    10.10.61.226   k8s-01   <none>           <none>
{{< / highlight >}}

DaemonSets and Deployments have a lot in common with a few essential differences. DaemonSets fill the use case of running a service all relevant nodes and auto-scale based on the number of relevant nodes within the cluster.