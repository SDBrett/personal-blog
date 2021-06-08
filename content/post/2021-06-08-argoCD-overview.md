---
author: Brett Johnson
categories:
- Kubernetes
- ArgoCD
date: "2021-06-08"
image: /assets/images/argo-horizontal-color.svg
tags:
- Kubernetes
- ArgoCD
title: ArgoCD - An overview
versions:
- software: Kubernetes
  versions:
  - v1.20
- software: ArgoCD
  versions:
  - v2.0.3
draft: false
---

![Argo](/assets/images/argo-horizontal-color.svg)


## What is ArgoCD

ArgoCD is a Continuous Delivery (CD) tool for Kubernetes, which applies manifests stored in Git repositories or Helm charts to Kubernetes clusters. Built on the GitOps model, ArgoCD uses the source repositories as the source of truth; this means that manifests and charts pulled from source repositories represent the true intended state. The GitOps model also means that the source control platform tracks changes to the desired state.

ArgoCD ensures configuration consistency through detection and remediation of drift between the clusters current state and the desired state. Maintaining configuration consistency helps to reduce operational effort and improve security across the environment.

Installing ArgoCD is well described in the [documentation](https://argo-cd.readthedocs.io/en/stable/) and straightforward. The documentation uses a command which applies the installation manifest directly from a GitHub link; this is ok in a lab environment but not a good practice in a real Kubernetes environment. 

You can configure ArgoCD before deployment by editing the install manifest, or for post-deployment configuration changes, edit ArgoCD Kubernetes objects.

## ArgoCD Kubernetes Objects

ArgoCD configuration settings are stored in several Kubernetes objects within the namespace that ArgoCD is deployed.

| File Name | Resource Name | Kind | Description |
|-----------|---------------|------|-------------|
| [`argocd-cm.yaml`](argocd-cm.yaml) | argocd-cm | ConfigMap | General Argo CD configuration |
| [`argocd-secret.yaml`](argocd-secret.yaml) | argocd-secret | Secret | Password, Certificates, Signing Key |
| [`argocd-rbac-cm.yaml`](argocd-rbac-cm.yaml) | argocd-rbac-cm | ConfigMap | RBAC Configuration |
| [`argocd-tls-certs-cm.yaml`](argocd-tls-certs-cm.yaml) | argocd-tls-certs-cm | ConfigMap | Custom TLS certificates for connecting Git repositories via HTTPS (v1.2 and later) |
| [`argocd-ssh-known-hosts-cm.yaml`](argocd-ssh-known-hosts-cm.yaml) | argocd-ssh-known-hosts-cm | ConfigMap | SSH known hosts data for connecting Git repositories via SSH (v1.2 and later) |
| [`application.yaml`](application.yaml) | - | Application | Example application spec |
| [`project.yaml`](project.yaml) | - | AppProject | Example project spec |

**[Table Source](https://argo-cd.readthedocs.io/en/stable/operator-manual/declarative-setup/#declarative-setup**)**


## Using ArgoCD

The ArgoCD web UI is very clean and provides some good information status information and limited configuration options. The `argocd` and `argocd-util` command-line interfaces are far more powerful than the web UI for managing ArgoCD. You can also manage ArgoCD using the Kubernetes resources listed in the table above.

## Cluster Management
ArgoCDs' ability to manage multiple Kubernetes clusters and revision targetting provides topology and multi-cluster management flexibility. You might choose to deploy ArgoCD on a single cluster, perhaps a dedicated management cluster, to provide a centralized cluster management service. ArgoCD automatically registers the cluster it's deployed on during the installation process.

**Centralized Management**

![Central Management](/assets/images/argocd-central.png)

Or you could deploy an instance of ArgoCD on each cluster, managing only the cluster that it's running on, creating a decentralized management solution. 

**Decentralized Management**

![Decentral Management](/assets/images/argocd-decentalized.png)

### Adding clusters

Clusters are added to ArgoCD using the command line utility; there is no option in the web UI. The below example uses credentials from ~/.kube/config for initial authentication. ArgoCD adds a service account to the cluster for future authentication.

{{% highlight shell %}}
$ argocd cluster add default/api-cluster-2590-dynamic-opentlc-com:6443/admin
INFO[0001] ServiceAccount "argocd-manager" created in namespace "kube-system"
INFO[0001] ClusterRole "argocd-manager-role" created
INFO[0001] ClusterRoleBinding "argocd-manager-role-binding" created
Cluster 'https://api.cluster-2590.dynamic.opentlc.com:6443' added
{{% / highlight %}}

{{% highlight shell %}}
$ kc -n kube-system get sa --context default/api-cluster-2590-dynamic-opentlc-com:6443/admin
NAME                                 SECRETS   AGE
argocd-manager                       2         103s
{{% / highlight %}}


### Projects

Projects are a logical construct to control application deployment and permissions to the resources of the application.
* Restrict source repositories
* Deployment destinations, clusters and namespaces
* What objects can be deployed
* Configuration application RBAC
<br>

Configuration of projects can be done in several ways:
* The ArgoCD web UI
* Configuring the `appprojects.argoproj.io` custom resource
* Using the ArgoCD command-line utility

### Applications

Applications contain the source repository to pull manifests or charts from, the target cluster and namespace and synchronization policies. Configuring applications is how you provide ArgoCD with the required information to apply the desired configuration and detect and remediate drift.

You can manage applications using several different methods:
* The ArgoCD web UI
* The `applications.argoproj.io`  Kubernetes custom resource
* The ArgoCD command-line utility

An application can only use one source repository, one project and one deployment target.

### Deploying an Application

I had trouble with synchronization failures while writing this post because ArgoCD was trying to create resources such as deployments and services accounts before namespaces and CRDS. I found that I needed to update the manifests by adding resource hooks to control the order in which ArgoCD created the resources. 

Make sure you read the documentation on resource hooks when learning ArgoCD. [Resource Hooks](https://argo-cd.readthedocs.io/en/stable/user-guide/resource_hooks/).

The first application deploys the Operator Lifecycle Manager, and the second application deploys the OpenShift Compliance Operator. 
The `prune` option means that ArgoCD will remove resources from the cluster if they no longer exist in the manifests.
The `selfHeal` option tells ArgoCD to remediate configuration drift back to the desired state automatically.

*Operator LifeCycle Manager*
{{% highlight yaml %}}
project: default
source:
  repoURL: 'https://github.com/SDBrett/operator-lifecycle-manager'
  path: deploy/upstream/manifests/0.18.1
  targetRevision: argo-test
destination:
  server: 'https://kubernetes.default.svc'
syncPolicy:
  automated:
    prune: true
    selfHeal: true
{{% / highlight %}}

*OpenShift Compliance Operator*

{{% highlight yaml %}}
project: default
source:
  repoURL: 'https://github.com/sdbrett/argocdgt'
  path: management/compliance
  targetRevision: HEAD
destination:
  server: 'https://kubernetes.default.svc'
syncPolicy:
  automated:
    prune: true
    selfHeal: true
{{% / highlight %}}

The web interface provides a nice topology of our application's resources.

![Compliance operator topology](/assets/images/compliance-operator-topology.png)


## Summary

In my opinion, removing the ability for operators to make changes directly is a crucial step to increasing configuration consistency (in any sized environment). Using CD tools such as ArgoCD ensures that all changes have passed through a validation process before implementation. The CI/CD process reduces the need for operators to make changes manually to a cluster.

ArgoCD appears to have all the features required to simplify multi-cluster management and a low barrier to entry. Other than getting caught out by resource hooks, I found ArgoCD easy to get running, and overall, the operational process of the platform makes sense.

Future posts will go into further detail about using ArgoCD and other solutions from Argo.