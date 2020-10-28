---
author: Brett Johnson
categories:
- Kubernetes
date: "2019-11-21T00:00:00Z"
image: /assets/images/Kubernetes_logo.svg
tags:
- Kubernetes
title: Kubernetes, Master Service Logging
versions:
- software: Kubernetes
  versions:
  - v1.15.x
  - v1.16.x
- software: Kubeadm
  versions:
  - v1.15.x
  - v1.16.x
---

I noticed that none of the Kubernetes services are writing logs to `/var/log/` on the master nodes. I built my lab with kubeadm and a reasonably basic configuration file, just the settings needed to make it work.

The configuration file contained just enough information to build the lab and make it work. The Kubernetes services  `apiServer`, `ControlManager` and `Scheduler` have several configuration flags for logging. 

In this post, I am going to go through my experience, enabling logging with kubeadm. The goal is to enable the services to write logs to `/var/log/`, not integration with a central logging system.

The below code block is the configuration file I used to build my lab cluster with kubeadm:

{{< highlight yaml >}}
apiVersion: kubeadm.k8s.io/v1beta2
kind: ClusterConfiguration
controlPlaneEndpoint: "k8s.sdbrett.lab:6443"
apiServer:
  certSANs:
  - "k8s.sdbrett.lab"
  - "192.168.17.40"
certificatesDir: /etc/kubernetes/pki
clusterName: kubernetes
dns:
  type: CoreDNS
imageRepository: k8s.gcr.io
kubernetesVersion: v1.16.0
networking:
  dnsDomain: cluster.local
  serviceSubnet: "172.21.0.0/24"
  podSubnet: "10.10.0.0/16"
{{< / highlight >}}

The [Command line reference][1] page provides additional arguments for each service. It appeared that I needed to add the `--log-dir` flag as an extra argument for each of the services and then run `sudo kubeadm init phase control-plane all --config=configfile.yaml` to generate new manifests for the ApiServer, ControllManager and Scheduler. The reality was not quite so simple, and I found a couple of sticking points to overcome.

Before continuing, I think it's important to mention that the Kubernetes master services are running as static pods on each of the master nodes. If you're not familiar with static pods; here's a post I prepared earlier: [Static Pods][2]

The first (and third) hurdle was making the logs available on the master and not just within the filesystem of the container. I could make this work by manually configuring the pod manifests, but, that doesn't teach the lessons I'm trying to learn. The 'How to update configurations with kubeadm', enabling logging to file is a secondary lesson. One attempt was adding `volumes` and `volumeMounts` to the configuration file, and the other was using `extraMounts`, these didn't work.

I found that I needed to upgrade my configuration file to one that supported the required options. My lab was built using kubeadm 1.15 (I think), and I have just updated it to 1.16. Meaning I was looking at documentation which didn't align to the configuration file version I currently had.

Upgrading the configuration file was a quick and painless effort, using the commands below.

 {{< highlight shell >}}
 $ mv config.yaml old-config.yml
 $ kubeadm config migrade --old-config old-config.yaml --new-config config.yaml
 {{< / highlight >}}

The config file now looks a lot different.

{{< highlight yaml >}}
apiVersion: kubeadm.k8s.io/v1beta2
bootstrapTokens:
- groups:
  - system:bootstrappers:kubeadm:default-node-token
  token: quwadb.ii0yt74tdvw6zebn
  ttl: 24h0m0s
  usages:
  - signing
  - authentication
kind: InitConfiguration
localAPIEndpoint:
  advertiseAddress: 192.168.17.43
  bindPort: 6443
nodeRegistration:
  criSocket: /var/run/dockershim.sock
  name: k8s-master-03
  taints:
  - effect: NoSchedule
    key: node-role.kubernetes.io/master
---
apiServer:
  certSANs:
  - k8s.sdbrett.lab
  - 192.168.17.40
  timeoutForControlPlane: 4m0s
apiVersion: kubeadm.k8s.io/v1beta2
certificatesDir: /etc/kubernetes/pki
clusterName: kubernetes
controlPlaneEndpoint: k8s.sdbrett.lab:6443
controllerManager: {}
dns:
  type: CoreDNS
etcd:
  local:
    dataDir: /var/lib/etcd
imageRepository: k8s.gcr.io
kind: ClusterConfiguration
kubernetesVersion: v1.16.0
networking:
  dnsDomain: cluster.local
  podSubnet: 10.10.0.0/16
  serviceSubnet: 172.21.0.0/24
scheduler: {}
{{< / highlight >}}

Now we are back at the problem of making the log files available on the local master nodes. This problem was solved by adding the below code below to the following sections in the new configuration file: `apiServer.extraVolumes`, `controllerManager.extraVolumes` and `scheduler.extraVolumes`.

{{< highlight yaml >}}
  - name: "var-log-kubernetes"
    hostPath: "/var/log/kubernetes"
    mountPath: "/var/log/kubernetes"
    readOnly: false
    pathType: Directory
{{< / highlight >}}

I then ran the following commands to test that the mounting and permissions worked.

{{< highlight shell >}}
k8s-master-01:$ sudo kubeadm init phase control-plane all --config.yaml
k8s-master-01:$ kubectl exec -it -n kube-system kube-apiserver-k8s-master-01 /bin/sh
kube-apiserver-k8s-master-01:$ touch /var/log/kubernetes/testFile
kube-apiserver-k8s-master-01:$ exit
k8s-master-01:$ ls /var/log/kubernetes/
test
{{< / highlight >}}

Things are looking promising, kubeadm is using the config file to generate manifests which provide an additional volume mount for log files.

The next part took quite some time to work through; getting the services actually to write the bloody logs to file. Like many things that take a long time, it seemed very easy at first glance. The task appeared to be `log-dir: /var/log/kubernetes` to the follow locations in the configuration file; `apiServer.extraArgs`, `controllerManager.extraArgs` and `scheduler.extraArgs`.

Kubeadm was used to generate new manifests, but, no-log files were appearing for any of the three services. I validated that the extra argument was in the manifests. An additional argument was required to stop logging to `stderr`; I added `logtostderr: false` to `apiServer.extraArgs`, `controllerManager.extraArgs` and `scheduler.extraArgs`.

This is where two things happened; kubeadm would no longer generate manifests, and I began to dig into the issue further (and cry crocodile tears, like saltwater crocodiles, not those itty bitty freshwater crocs). 

{{< highlight shell >}}
$ sudo kubeadm init phase control-plane all --config=config.yaml
I1121 12:46:41.842430  115767 initconfiguration.go:190] loading configuration from "config.yaml"
W1121 12:46:41.844801  115767 strict.go:54] error unmarshaling configuration schema.GroupVersionKind{Group:"kubeadm.k8s.io", Version:"v1beta2", Kind:"ClusterConfiguration"}: error converting YAML to JSON: yaml: unmarshal errors:
  line 28: key "apiVersion" already set in map
v1beta2.ClusterConfiguration.APIServer: v1beta2.APIServer.ControlPlaneComponent: ExtraArgs: ReadString: expects " or n, but found t, error found in #10 byte of ...|ostderr":true,"log-d|..., bigger context ...|,"192.168.17.40"],"extraArgs":{"alsologtost
derr":true,"log-dir":"/var/log/kubernetes"},"extraVolumes|.
{{< / highlight >}}

Manually adding the argument to the manifest worked a treat and logs began to appear on the master node. But, like the task of mounting volumes, it doesn't achieve the desired objective.

After quite some time of searching, I stumbled across a response to an arbitrary issue on GitHub; kubeadm expects all extraArgs values to be strings. In YAML, a value of 'true' or 'false' is considered as type boolean unless wrapped in quotation marks. Kubeadm generated manifests and logs were being written to /var/log/kubernetes on the master node. 

As this cluster is a multi-master cluster, the config file needs to be copied to the other master nodes and kubeadm run locally on them to generate new manifests. Below is the complete configuration file.

{{< highlight yaml >}}

apiVersion: kubeadm.k8s.io/v1beta2
bootstrapTokens:
- groups:
  - system:bootstrappers:kubeadm:default-node-token
  token: ax9vy6.12kxk8vwutmm8wy4
  ttl: 24h0m0s
  usages:
  - signing
  - authentication
kind: InitConfiguration
localAPIEndpoint:
  advertiseAddress: 192.168.17.41
  bindPort: 6443
nodeRegistration:
  criSocket: /var/run/dockershim.sock
  name: k8s-master-01
  taints:
  - effect: NoSchedule
    key: node-role.kubernetes.io/master
---
apiServer:
  certSANs:
  - k8s.sdbrett.lab
  - 192.168.17.40
  extraArgs:
    log-dir: /var/log/kubernetes
    logtostderr: "true"
  timeoutForControlPlane: 4m0s
  extraVolumes:
  - name: "var-log-kubernetes"
    hostPath: "/var/log/kubernetes"
    mountPath: "/var/log/kubernetes"
    readOnly: false
    pathType: Directory
apiVersion: kubeadm.k8s.io/v1beta2
certificatesDir: /etc/kubernetes/pki
clusterName: kubernetes
controlPlaneEndpoint: k8s.sdbrett.lab:6443
controllerManager:
  extraArgs:
    log-dir: /var/log/kubernetes
    logtostderr: "false"
  extraVolumes:
  - name: "var-log-kubernetes"
    hostPath: "/var/log/kubernetes"
    mountPath: "/var/log/kubernetes"
    readOnly: false
    pathType: Directory
apiVersion: kubeadm.k8s.io/v1beta2
dns:
  type: CoreDNS
etcd:
  local:
    dataDir: /var/lib/etcd
imageRepository: k8s.gcr.io
kind: ClusterConfiguration
kubernetesVersion: v1.16.0
networking:
  dnsDomain: cluster.local
  podSubnet: 10.10.0.0/16
  serviceSubnet: 172.21.0.0/24
scheduler:
  extraArgs:
    log-dir: /var/log/kubernetes
    logtostderr: "false"
  extraVolumes:
  - name: "var-log-kubernetes"
    hostPath: "/var/log/kubernetes"
    mountPath: "/var/log/kubernetes"
    readOnly: false
    pathType: Directory
{{< / highlight >}}

#### References:

- [Command line tools reference][3]
- [Kubeadm init][4]
- [Kubeadm config][5]


[1]: https://kubernetes.io/docs/reference/command-line-tools-reference
[2]: {{< ref "post/2019-11-20-kubernetes-static-pods" >}}
[3]: https://kubernetes.io/docs/reference/command-line-tools-reference/kube-apiserver/
[4]: https://kubernetes.io/docs/reference/setup-tools/kubeadm/kubeadm-init/
[5]: https://kubernetes.io/docs/reference/setup-tools/kubeadm/kubeadm-config/