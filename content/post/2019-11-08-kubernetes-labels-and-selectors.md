---
author: Brett Johnson
categories:
- Kubernetes
date: "2019-11-08T00:00:00Z"
image: /assets/images/Kubernetes_logo.svg
tags:
- Kubernetes
title: Kubernetes, Labels and Selectors
versions:
- software: Kubernetes
  versions:
  - v1.15.x
---

Labels are a key/value formatted peice of metadata attached to an object within Kubernetes. Labels provide additional information about the object with relevance to the consumer or object. For example, a label can specify hardware characteristics of a node or if a workload is for testing of production.

Labels implicitly group like objects together while providing a lookup mechanism for users, controllers and external systems.

For this example, I have a deployment for a web service which runs on internal infrastructure and one which runs on DMZ infrastructure.

{{< highlight yaml >}} 
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web-internal
spec:
  replicas: 4
  selector:
    matchLabels:
      run: nginx
      location: internal
  template:
    metadata:
      labels:
        run: nginx
        location: internal
    spec:
      containers:
      - image: nginx
        name: nginx
{{< / highlight >}}

{{< highlight yaml >}} 
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-dmz
spec:
  replicas: 4
  selector:
    matchLabels:
      run: nginx
      location: dmz
  template:
    metadata:
      labels:
        run: nginx
        location: dmz
    spec:
      containers:
      - image: nginx
        name: nginx
{{< / highlight >}}

The labels `run=nginx` and either `location=dmz` or `location=internal` are on the pods at deployment. An object can have multiple labels, but an object cannot have multiple labels with the same key. If an objects manifest has multiple label entries with the same key, the last value is applied.

The below code block shows the pods deployed by applying the above deployment manifests.

{{< highlight bash >}}
$ kubectl get pods --show-labels
NAME                            READY   STATUS    RESTARTS   AGE   LABELS
nginx-dmz-57cdc7848c-9bsft      1/1     Running   0          18h   location=dmz,pod-template-hash=57cdc7848c,run=nginx
nginx-dmz-57cdc7848c-l7clm      1/1     Running   0          18h   location=dmz,pod-template-hash=57cdc7848c,run=nginx
nginx-dmz-57cdc7848c-ngcqj      1/1     Running   0          18h   location=dmz,pod-template-hash=57cdc7848c,run=nginx
nginx-dmz-57cdc7848c-s9h6l      1/1     Running   0          18h   location=dmz,pod-template-hash=57cdc7848c,run=nginx
web-internal-6b444b6dd9-kd8w2   1/1     Running   0          18h   location=internal,pod-template-hash=6b444b6dd9,run=nginx
web-internal-6b444b6dd9-mxjqz   1/1     Running   0          18h   location=internal,pod-template-hash=6b444b6dd9,run=nginx
web-internal-6b444b6dd9-vm7hn   1/1     Running   0          18h   location=internal,pod-template-hash=6b444b6dd9,run=nginx
web-internal-6b444b6dd9-zwvp9   1/1     Running   0          18h   location=internal,pod-template-hash=6b444b6dd9,run=nginx
{{< / highlight >}}

*NOTE:* The deployment controller automatically applies the `pod-template-hash` label.

Adding one or more `--selector` options to `kubectl get pods` filters the pods returned.

{{< highlight bash >}}
$ kubectl get pods --selector run=nginx --show-labels
NAME                            READY   STATUS    RESTARTS   AGE   LABELS
nginx-dmz-57cdc7848c-9bsft      1/1     Running   0          18h   location=dmz,pod-template-hash=57cdc7848c,run=nginx
nginx-dmz-57cdc7848c-l7clm      1/1     Running   0          18h   location=dmz,pod-template-hash=57cdc7848c,run=nginx
nginx-dmz-57cdc7848c-ngcqj      1/1     Running   0          18h   location=dmz,pod-template-hash=57cdc7848c,run=nginx
nginx-dmz-57cdc7848c-s9h6l      1/1     Running   0          18h   location=dmz,pod-template-hash=57cdc7848c,run=nginx
web-internal-6b444b6dd9-kd8w2   1/1     Running   0          18h   location=internal,pod-template-hash=6b444b6dd9,run=nginx
web-internal-6b444b6dd9-mxjqz   1/1     Running   0          18h   location=internal,pod-template-hash=6b444b6dd9,run=nginx
web-internal-6b444b6dd9-vm7hn   1/1     Running   0          18h   location=internal,pod-template-hash=6b444b6dd9,run=nginx
web-internal-6b444b6dd9-zwvp9   1/1     Running   0          18h   location=internal,pod-template-hash=6b444b6dd9,run=nginx
{{< / highlight >}}

Use multiple selectors to get internal nginx pods.

{{< highlight bash >}}
$ kubectl get pods --selector run=nginx --selector location=internal --show-labels 
NAME                            READY   STATUS    RESTARTS   AGE   LABELS
web-internal-6b444b6dd9-kd8w2   1/1     Running   0          18h   location=internal,pod-template-hash=6b444b6dd9,run=nginx
web-internal-6b444b6dd9-mxjqz   1/1     Running   0          18h   location=internal,pod-template-hash=6b444b6dd9,run=nginx
web-internal-6b444b6dd9-vm7hn   1/1     Running   0          18h   location=internal,pod-template-hash=6b444b6dd9,run=nginx
web-internal-6b444b6dd9-zwvp9   1/1     Running   0          18h   location=internal,pod-template-hash=6b444b6dd9,run=nginx
{{< / highlight >}}

The deployment controller uses a selector to identify which pods are part of a deployment. Creating a deployment also creates a replicaSet object which is monitored by the replication controller to ensure that the number of pods running matches the desired number.

Right now our deployments have 4 pods running in a ready state.

{{< highlight bash >}}
$ kubectl get deployments
NAME           READY   UP-TO-DATE   AVAILABLE   AGE
nginx-dmz      4/4     4            4           23h
web-internal   4/4     4            4           23h
{{< / highlight >}}

{{< highlight bash >}}
$ kubectl get rs
NAME                      DESIRED   CURRENT   READY   AGE
nginx-dmz-57cdc7848c      4         4         4       23h
web-internal-6b444b6dd9   4         4         4       23h
{{< / highlight >}}

Running `kubectl get rs -o yaml` displays the configuration of a replicaSet

{{< highlight yaml >}}
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  annotations:
    deployment.kubernetes.io/desired-replicas: "4"
    deployment.kubernetes.io/max-replicas: "5"
    deployment.kubernetes.io/revision: "1"
  creationTimestamp: "2019-11-07T02:31:10Z"
  generation: 1
  labels:
    location: internal
    pod-template-hash: 6b444b6dd9
    run: nginx
  name: web-internal-6b444b6dd9
  namespace: default
  ownerReferences:
  - apiVersion: apps/v1
    blockOwnerDeletion: true
    controller: true
    kind: Deployment
    name: web-internal
    uid: 0de8b806-c877-482d-90fb-186242024c91
  resourceVersion: "279401"
  selfLink: /apis/apps/v1/namespaces/default/replicasets/web-internal-6b444b6dd9
  uid: 3dd114b0-9005-42ee-9218-05901a816144
spec:
  replicas: 4
  selector:
    matchLabels:
      location: internal
      pod-template-hash: 6b444b6dd9
      run: nginx
  template:
    metadata:
      creationTimestamp: null
      labels:
        location: internal
        pod-template-hash: 6b444b6dd9
        run: nginx
    spec:
      containers:
      - image: nginx
        imagePullPolicy: Always
        name: nginx
        resources: {}
        terminationMessagePath: /dev/termination-log
        terminationMessagePolicy: File
      dnsPolicy: ClusterFirst
      restartPolicy: Always
      schedulerName: default-scheduler
      securityContext: {}
      terminationGracePeriodSeconds: 30
status:
  availableReplicas: 4
  fullyLabeledReplicas: 4
  observedGeneration: 1
  readyReplicas: 4
  replicas: 4
{{< / highlight >}}

Lets adust one of the labels used on a pod, so it no longer matches our `selector.matchLabels` spec and see what happens.

{{< highlight bash >}}
$ kubectl label pod web-internal-6b444b6dd9-kd8w2 run=notnginx --overwrite
$ kubectl get pods --selector location=internal --show-labels 
$ kubectl get deployments
NAME           READY   UP-TO-DATE   AVAILABLE   AGE
nginx-dmz      4/4     4            4           23h
web-internal   3/4     4            3           23h

$kubectl get pods --selector location=internal --show-labels 
NAME                            READY   STATUS    RESTARTS   AGE   LABELS
web-internal-6b444b6dd9-kd8w2   1/1     Running   0          23h   location=internal,pod-template-hash=6b444b6dd9,run=notnginx
web-internal-6b444b6dd9-mxjqz   1/1     Running   0          23h   location=internal,pod-template-hash=6b444b6dd9,run=nginx
web-internal-6b444b6dd9-vm7hn   1/1     Running   0          23h   location=internal,pod-template-hash=6b444b6dd9,run=nginx
web-internal-6b444b6dd9-wbdvc   1/1     Running   0          35s   location=internal,pod-template-hash=6b444b6dd9,run=nginx
web-internal-6b444b6dd9-zwvp9   1/1     Running   0          23h   location=internal,pod-template-hash=6b444b6dd9,run=nginx$ 
{{< / highlight >}}

The replica controller could only find 3 pods running using the matchLabel selector after we changing the `run` label on pod `web-internal-6b444b6dd9-kd8w2` and deployed another pod to remediate the issue. 

Labels and selectors have many additional uses to the one covered in this post; they are a core part of the Kubernetes platform.