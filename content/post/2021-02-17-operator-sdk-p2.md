---
author: Brett Johnson
postseries: operators
categories:
- Kubernetes
- Operators
date: "2021-02-17"
image: /assets/images/operator_logo_sdk_color.svg
tags:
- Kubernetes
- Operators
title: Operator SDK Part 2
versions:
- software: Kubernetes
  versions:
  - v1.17.x
  - v1.18.x
  - v1.19.x
draft: false
---

[*Feature image source*](https://raw.githubusercontent.com/operator-framework/operator-sdk/master/website/static/operator_logo_sdk_color.svg)

This post continues from [Operator SDK Part1]({{< ref "2021-02-08-operator-sdk-p1.md" >}}) and looks at building the operators container image and running the operator.

## Building the Operator Container

An operator container image contains the operator runtime for the type of operator and your business logic.

The command `make docker-build IMG=$IMG_NAME:TAG` creates the operator container image using a `Dockerfile` in the projects root directory. The initialization process generates the `Dockerfile` based on the type of operator.

The image needs to be pushed to an image registry before Kubernetes clusters can consume it.

*Dockerfile for Ansible Operator*
{{% highlight docker %}}
FROM quay.io/operator-framework/ansible-operator:v1.3.0

COPY requirements.yml ${HOME}/requirements.yml
RUN ansible-galaxy collection install -r ${HOME}/requirements.yml \
 && chmod -R ug+rwx ${HOME}/.ansible

COPY watches.yaml ${HOME}/watches.yaml
COPY roles/ ${HOME}/roles/
COPY playbooks/ ${HOME}/playbooks/
{{% / highlight %}}


*Dockerfile for Helm Operator*
{{% highlight docker %}}
# Build the manager binary
FROM quay.io/operator-framework/helm-operator:v1.2.0

ENV HOME=/opt/helm
COPY watches.yaml ${HOME}/watches.yaml
COPY helm-charts  ${HOME}/helm-charts
WORKDIR ${HOME}
{{% / highlight %}}

*Dockerfile for Helm Operator*
{{% highlight docker %}}
FROM golang:1.15 as builder

WORKDIR /workspace
# Copy the Go Modules manifests
COPY go.mod go.mod
COPY go.sum go.sum
# cache deps before building and copying source so that we don't need to re-download as much
# and so that source changes don't invalidate our downloaded layer
RUN go mod download

# Copy the go source
COPY main.go main.go
COPY api/ api/
COPY controllers/ controllers/

# Build
RUN CGO_ENABLED=0 GOOS=linux GOARCH=amd64 GO111MODULE=on go build -a -o manager main.go

# Use distroless as minimal base image to package the manager binary
# Refer to https://github.com/GoogleContainerTools/distroless for more details
FROM gcr.io/distroless/static:nonroot
WORKDIR /
COPY --from=builder /workspace/manager .
USER 65532:65532

ENTRYPOINT ["/manager"]
{{% / highlight %}}


The operator container runs on a cluster in a pod called the `manager`. The manager's default `imagePullPolicy` is `IfNotPresent`, which means if you rerun a test with the same image tag, the cluster does not pull the updated image. To counter this behaviour without changing the `imagePullPolicy` I use separate repositories for testing and production images.

Testing image tag: &emsp; quay.io/brejohns/vault-helm-dev:0.0.50
Published image tag: &emsp; quay.io/brejohns/vault-helm:0.0.2

## Create a Bundle

Bundle creation uses Kustomize to generate manifests from the files in the `config/` directory. The bundle includes
A bundle contains all the manifests required to deploy your operator to a cluster. 

There are several arguments required to generate a bundle using the `Makefile`:
- `CHANNELS` defines the operators release channels
- `DEFAULT_CHANNEL` the default channel to use if none is specified when deploying the operator
- `VERSION` the operators version number
- `IMG` the operators container image tag

After creating the bundle manifests can build and push a bundle container image. This image contains the bundle manifests and is used by OLM.

## Deploying an Operator

There are several options deployment options available; Local, Bundle and OLM. You are likely to use all three during various operator testing and development stages, though OLM is the most like to be used for final distribution.

All deployment options require that the `current-content` in `~/.kube/config` or the file referenced by the environmental variable `KUBECONFIG` is valid.

**Local**

Run the command `make install kustomize` to generate and apply the required manifests to the cluster.

The command `make install run` applies all your manifests to the cluster and runs the manager process on your local machine. The command obtains the operator runtime binary for Ansible and Helm based operators; a binary is compiled for a Go-based operator.

Running an operator is simple deployment method and helpful for testing during the development process as the manager logs stream to `stdout`.

**As a Deployment**

This option deploys the operator to the cluster using the manifest template `config/manager/manager.yaml`.

To deploy the operator as a deployment, you'll need to build the operators container image and push it to an image registry. Additional steps may be required to configure image pull secrets if you're using a private registry.

Make sure to change the tag number each time or set the `imagePullPolicy` to `always` in `config/manager/manager.yaml`.

After completing the image build and push steps, run `make deploy IMG=<image tag>`. This command generates manifests with Kustomize and then applies them to the cluster.

Run `make undeploy` to clean up the cluster between tests.

**OLM**

This method uses the OLM to deploy and manage your operator on a cluster by running an operator registry on your cluster. You'll need to complete several steps to deploy using OLM.


Check if OLM is installed on the cluster by running `operator-SDK olm status`.

*OLM not installed:*

{{% highlight shell %}}
$ operator-sdk olm status
I0217 11:10:20.223004   65134 request.go:645] Throttling request took 1.048103206s, request: GET:https://192.168.99.103:8443/apis/extensions/v1beta1?timeout=32s
FATA[0002] Failed to get OLM status: error getting installed OLM version (set --version to override the default version): no existing installation found 
{{% / highlight %}}

*OLM installed:*

{{% highlight shell %}}
$ operator-sdk olm status
INFO[0000] Fetching CRDs for version "0.17.0"           
INFO[0000] Using locally stored resource manifests      
INFO[0000] Successfully got OLM status for version "0.17.0" 

NAME                                            NAMESPACE    KIND                        STATUS
operators.operators.coreos.com                               CustomResourceDefinition    Installed
operatorgroups.operators.coreos.com                          CustomResourceDefinition    Installed
installplans.operators.coreos.com                            CustomResourceDefinition    Installed
clusterserviceversions.operators.coreos.com                  CustomResourceDefinition    Installed
olm-operator                                    olm          Deployment                  Installed
subscriptions.operators.coreos.com                           CustomResourceDefinition    Installed
olm-operator-binding-olm                                     ClusterRoleBinding          Installed
operatorhubio-catalog                           olm          CatalogSource               Installed
olm-operators                                   olm          OperatorGroup               Installed
aggregate-olm-view                                           ClusterRole                 Installed
catalog-operator                                olm          Deployment                  Installed
aggregate-olm-edit                                           ClusterRole                 Installed
olm                                                          Namespace                   Installed
global-operators                                operators    OperatorGroup               Installed
operators                                                    Namespace                   Installed
package server                                   olm          ClusterServiceVersion       Installed
olm-operator-serviceaccount                     olm          ServiceAccount              Installed
catalogsources.operators.coreos.com                          CustomResourceDefinition    Installed
system:controller:operator-lifecycle-manager                 ClusterRole                 Installed
{{% / highlight %}}

Run `operator-sdk olm install` to install OLM if it is not already installed.

To deploy with OLM
- Build and Deploy an operator container image
- Create bundle manifests
- Build and push bundle image
- Validate bundle
- Use operator-SDK to serve the bundle to OLM.

The below commands perform the above-mentioned tasks:

{{% highlight shell %}}
VERSION=0.0.1
IMAGE_REGISTRY=''
IMG=$IMAGE_REGISTRY:$VERSION
BUNDLE_IMAGE=$IMAGE_REGISTRY-bundle:$VERSION
BUNDLE_CHANNELS=alpha
OPERATOR_NAMESPACE=my-operator

kubectl create ns $OPERATOR_NAMESPACE

# Build and push the operator container image
make docker-build IMG=$IMG
make docker-push IMG=$IMG

# Make bundle
make bundle CHANNELS=$BUNDLE_CHANNELS DEFAULT_CHANNEL=$BUNDLE_CHANNELS VERSION=$VERSION IMG=$IMG 

# Build and push the bundle container image
make bundle-build BUNDLE_IMG=$BUNDLE_IMAGE
make docker-push IMG=$BUNDLE_IMAGE

# Validate bundle
operator-SDK bundle validate $BUNDLE_IMAGE

# Serve bundle image to OLM
operator-sdk run bundle $BUNDLE_IMAGE --namespace $OPERATOR_NAMESPACE
{{% /highlight %}}

The `operator-sdk run bundle` command deploys a pod that serves as your operators' source catalogue and creates a `subscription` object. A `subscription` object specifies your intent to install and operator and describes the installation parameters.

## Summary

Part 2 of our look into the Operator-SDK focused on how to build and deploy operators for testing. This post did not cover distribution and Deployment using operator registries; a future post will cover this topic.

I hope this post helps you to continue building and deploying operators of your own.


