---
author: Brett Johnson
categories:
- Kubernetes
date: "2019-11-20T00:00:00Z"
image: /assets/images/Kubernetes_logo.svg
tags:
- Kubernetes
title: Kubernetes, Static Pods
versions:
- software: Kubernetes
  versions:
  - v1.15.x
---

Static pods are like regular pods, except they are managed by the `kubelet` service on a node and not the API server. The `kubelet` creates a `mirror pod` on the API server, which is a read-only copy that allows the pod to be seen on the API server but not controlled.

The `kubelet` service is responsible for managing the state of the pod and controlling restarts, for example.

The `kubelet` process watches the location specified with the configuration parameter `--manifest-url` or `--pod-manifest-path`.

There are several ways which you can view the configuration of a node. The examples below are based on a default deployment using `kubeadm`, where `kubelet` runs as a systemd service.

I am going to use the node `k8s-01` for the below examples.

**Using the API Server**

{{< highlight shell >}}
$ kubectl proxy #By default this will use port 8001
$ curl -sSL "http://localhost:8001/api/v1/nodes/k8s-01/proxy/configz"
{{< / highlight >}}

**Configuration file**

{{< highlight shell >}}
$ ssh user@k8s-01.sdbrett.lab
$ systemctl status kubelet
# The output will provide the run commands, the --config flag specifies the location of config.yaml. In this case the path is /var/lib/kubelet/config.yaml
$ sudo cat /var/lib/kubelet/config.yaml
{{< / highlight >}}

**NOTE:** that the default permissions on /var/lib/kubelet are 700, so `sudo` or root is required to read the config file as well as modify.

Using ` | grep static` makes it easier to find the entry in both examples.

The `staticPodPath` entry in `config.yaml` overwrites the `--manifest-url` or `--pod-manifest-path` arguments in the systemd service file. You need to comment out or remove the `staticPodPath` entry in `config.yaml` if you wish to use arguments in the service file.

Creating a static pod is as simple as placing a manifest in the correct location to be read by the `kubelet`. You can do this manually, through using a configuration management solution and version control would be a better option for production environments.

{{< highlight shell >}}
$ ssh user@k8s-01.sdbrett.lab
$ sudo cat > /etc/kubernetes/manifests/static-nginx.yaml <<EOF
apiVersion: v1
kind: Pod
metadata:
  name: nginx
  labels:
    env: test
spec:
  containers:
  - name: nginx
    image: nginx
    imagePullPolicy: IfNotPresent
EOF
{{< / highlight >}}

You can see the static pod running with `kubectl`.

{{< highlight shell >}}
$kubectl get pods
NAME           READY   STATUS    RESTARTS   AGE
nginx-k8s-01   1/1     Running   0          14m
{{< / highlight >}}

Deleting the manifest file removes the pod.

Static pods are suitable for when you need pods on some specific nodes but not all. If you need a pod on every node, then use a DaemonSet. If you need to ensure pods are on different nodes, then have a look at anti-affinity rules.
