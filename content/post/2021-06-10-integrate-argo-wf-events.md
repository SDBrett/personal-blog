---
author: Brett Johnson
categories:
- Kubernetes
- Argo
date: "2021-06-08"
image: /assets/images/argo-horizontal-color.svg
tags:
- Kubernetes
- Argo
title: Integrating Argo Workflows and Events
versions:
- software: Kubernetes
  versions:
  - v1.20
- software: Argo Workflows
  versions:
  - v2.0.3
- software: Argo Events
  versions:
  - v2.0.3
draft: false
---

Argo Events and Workflows are two different and independent solutions from the Argo project; Events is an event driven automation framework and Workflows is a job orchestration engine. The purpose of this post is to demonstrate how to acheive event driven workflow execution by using Events and Workflows. Event based achitecutres are a key part of building solutions where the individual components are decoupled, the line of responsibility for the action which created an event stops at point of generating the event, what happens next is a problem for another system.

*Note: I am working on a post which dives depper into event driven architectures and asynchronisity which I will link here when done*

This post will focus on the specifically on the features of Events and Workflows relevant to integrating Argo Events and Argo Workflows

# Core Components

This section provides a quick overview of the core components for Argo Events and Argo Workflows. All of these components are Kubernetes resources which are consumed to deploy and configure additional resources on the cluster such as deployments.

## Argo Events
<Arch Image>
**Event Source** 
Event Sources specify how to consume events from external services such as; GitHub, Slack and Webhooks. There are 23 different event sources available at the time of writting.

An Event Source must be configured to use a service account with `[List, Watch]` permissions to monitor Kubernetes resources.

**Event Bus**
An Event Bus is a transport service for events to get from an event source to a sensor. 

**Sensors**
Sensors specify the filtering criteria and triggers. The sensor ingests events on the bus, checking them against the the filtering criteria, a response is triggered when an event matches.

There are currently 10 predefined triggers for services such as; HTTP requests, Kubernetes Objects and Slack. Additionally, you can create your own triggers as needed.

A service account with sufficient permissions needs to be provided if the trigger manipulates Kubernetes Objects.

## Argo Workflows

**Workflow Templates**
Common tasks performed by workflows can be defined as Workflow Templates and consumed by workflows to reduce repetition.

**Workflows**
A `workflows/argoproj.io/v1alpha1` resource defines the tasks to be executed by a workflow and intent to execute the workflow. Workflow execution is immediate and a once off event. Use the  `cronWorkflows/argoproj.io/v1alpha1` resource to create workflows with delayed execution or execute on a recurring schedule.

**Workflow Controller**

The Workflow Controller watches the Kubernetes API server for `workflows/argoproj.io/v1alpha1` and `cronWorkflows/argoproj.io/v1alpha1` resources. You specify the controllers scope to either it's own namespace or the entire cluster using the Argo Workflows deployment manifest.  A cluster wide scope is required for this integration to work as the Workflow Controller needs to observe the creation of `workflows/argoproj.io/v1alpha1` resources in the argo-events namespace,0.

The Workflow Controller executes a workflow by creating a `executor pod` in which the workflow process runs. The executor pod authenticates against with the Kubeneretes API server using the service account specified in the pods configuration. The `default` service account of the namespace which the executor pod is running in is used if the workflows `spec.serviceAccountName` field is empty.

**Argo Workflow Deployment**

The steps below detail how to deploy Argo Workflows.

{{% highlight shell %}}
$ kubectl create ns argo
$ kubectl -n argo apply -f https://raw.githubusercontent.com/argoproj/argo-workflows/stable/manifests/install.yaml
{{% / highlight %}}

After the application is running, if you're on using a local cluster like Minikube run this command so you can access the web interface.

{{% highlight shell %}}
$ kubectl -n argo port-forward deployment/argo-server 2746:2746 &
{{% / highlight %}}

The web interface address is https://localhost:2746

**Example Workflow** 

Or example workflow creates a pod running the `docker/whalesay:latest` image.

{{% highlight yaml % }}
apiVersion: argoproj.io/v1alpha1
kind: Workflow
metadata:
  generateName: webhook-
spec:
  serviceAccountName: workflow-pods-sa
  entrypoint: whalesay
  arguments:
    parameters:
    - name: message
      value: hello world
  templates:
  - name: whalesay
    inputs:
      parameters:
      - name: message
    container:
      image: docker/whalesay:latest
      command: [cowsay]
      args: ["{{inputs.parameters.message}}"]
{{ / highlight }}


## Argo Events Deployment

The commands below deploy Argo Events controllers and the event bus service.

{{% highlight shell %}}
$ kubectl create ns argo-events
# Install Argo Events
$ kubectl apply -f https://raw.githubusercontent.com/argoproj/argo-events/stable/manifests/install.yaml

# Deploy the Event Bus
$ kubectl apply -n argo-events -f https://raw.githubusercontent.com/argoproj/argo-events/stable/examples/eventbus/native.yaml
{{% / highlight %}}

# Integration Example

Argo Events and Argo Workflows have been deployed on a minikube cluster using the commands above. Our use case for this example is to deploy a pod based on a HTTP request from an external source.

The diagram below is the user level view of our example workflow, they submit a request and a workload is deployed.

This diagram details the data and process flow or our example workflow. Notice that Argo Events and Argo Workflows are completely decoupled.

**Event Source**
Our workflow needs to be able to receive HTTP Requests from the external system, this is acheived using an WebHook EventSource. 

{{% highlight yaml %}}
apiVersion: argoproj.io/v1alpha1
kind: EventSource
metadata:
  name: webhook
  namespace: argo-events
spec:
  service:
    ports:
    - port: 12000
      targetPort: 12000
  webhook: 
    example:
      endpoint: /deploy
      method: POST
      port: "12000"
      url: ""
{{% / highlight %}}

This manifest tells the `EventSourceController` to:
* Create a deployment with port 12000 exposed
* Create a service which forwards port 12000 to the eventsource pod on port 12000

Write events matching the follow criteria to the EventBus:
* Received on port 12000
* URI endpoint is /example
* Method is POST
* Any server URL

{{% highlight shell %}}
$ kubectl -n argo-events apply -f event-source.yaml
# Port forward command because this is being run on a Minikube cluster
$ kubectl -n argo-events port-forward $(kubectl -n argo-events get pod -l eventsource-name=webhook -o name) 12000:12000 &
# Test Webhook
$ curl -d '{"message":"Dont call me Shirley"}' -H "Content-Type: application/json" -X POST http://localhost:12000/deploy
Handling connection for 12000
success
# Validate writing to event bus
$ kubectl -n argo-events logs webhook-eventsource-n6g9n-d5c8f7985-s8msz
{
    "level": "info",
    "ts": 1623380073.806351,
    "logger": "argo-events.eventsource",
    "caller": "eventsources/eventing.go:411",
    "msg": "succeeded to publish an event",
    "eventSourceName": "webhook",
    "eventName": "example",
    "eventSourceType": "webhook",
    "eventID": "66326465363962312d623431662d343732332d396565302d643061643065323437303032"
} 
{{% / highlight %}}

At this stage our solution is able to receive a HTTP request and write to the EventBus.

## RBAC
Our integration example manipulates Kubernetes resources, therefore we need to create service accounts with the correct access rights to succeed. 

The sensor requires a service account to create a `workflows/argoproj.io/v1alpha1` object in the `argo-events` namespace. The manifest below creates the service account, role and role binding. The role does grant a few more permissions than explicitly required, but offers more flexibility for the types of workflows it can manage.

{{% highlight yaml %}}
apiVersion: v1
kind: ServiceAccount
metadata:
  namespace: argo-events
  name: operate-workflow-sa
---
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: operate-workflow-role
  namespace: argo-events
rules:
  - apiGroups:
      - argoproj.io
    verbs:
      - "*"
    resources:
      - workflows
      - workflowtemplates
      - cronworkflows
      - clusterworkflowtemplates
---
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: operate-workflow-role-binding
  namespace: argo-events
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: Role
  name: operate-workflow-role
subjects:
  - kind: ServiceAccount
    name: operate-workflow-sa
    namespace: argo-events
{{% / highlight %}}


This example workflow runs a container image on the Kubernetes cluster, therefore the executor pod needs a service account with+ permissions to create pods.

Below is the manifest for the service account used by the executor pod and the role and rolebindings.

{{% highlight yaml % }}
apiVersion: v1
kind: ServiceAccount
metadata:
  namespace: argo-events
  name: workflow-pods-sa
---
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: workflow-pods-role
rules:
  - apiGroups:
      - ""
    verbs:
      - "*"
    resources:
      - pods
---
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: workflow-pods-role-binding
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: Role
  name: workflow-pods-role
subjects:
  - kind: ServiceAccount
    name: workflow-pods-sa
    namespace: argo-events
{{ / highlight }}

## Sensor

The manifest below specifies the configuration for our Sensor. [Manifest source](https://raw.githubusercontent.com/argoproj/argo-events/stable/examples/sensors/webhook.yaml)

{{% highlight yaml %}}
apiVersion: argoproj.io/v1alpha1
kind: Sensor
metadata:
  name: webhook
spec:
  template:
    serviceAccountName: operate-workflow-sa
  dependencies:
    - name: test-dep
      eventSourceName: webhook
      eventName: example
  triggers:
    - template:
        name: webhook-workflow-trigger
        k8s:
          group: argoproj.io
          version: v1alpha1
          resource: workflows
          operation: create
          source:
            resource:
              apiVersion: argoproj.io/v1alpha1
              kind: Workflow
              metadata:
                generateName: webhook-
              spec:
                entrypoint: whalesay
                arguments:
                  parameters:
                  - name: message
                    # the value will get overridden by event payload from test-dep
                    value: hello world
                templates:
                - name: whalesay
                  inputs:
                    parameters:
                    - name: message
                  container:
                    image: docker/whalesay:latest
                    command: [cowsay]
                    args: ["{{inputs.parameters.message}}"]
          parameters:
            - src:
                dependencyName: test-dep
              dest: spec.arguments.parameters.0.value
{{% / highlight %}}

Apply the Sensor resource.

{{% highlight shell %}}
$ kubectl -n argo-events apply webhook-sensor.yaml
# The sensor controller creates a deployment for the sensor
$ kubectl  -n argo-events get deployments -l sensor-name=webhook
NAME                   READY   UP-TO-DATE   AVAILABLE   AGE
webhook-sensor-2hgxx   1/1     1            1           12s
{{% / highlight %}}

# End to end testing

All the peices should be in place so our users can execute a workflow from an external system. Each item has been tested individually and we now need to validate the end to end process by sending a HTTP Request to the event source using curl.

{{% highlight shell %}}
$ curl -d '{"message":"Dont call me Shirley"}' -H "Content-Type: application/json" -X POST http://localhost:12000/deploy
$ kubectl -n argo-events get workflows.argoproj.io
NAME            STATUS    AGE
webhook-l7zsp   Running   8s 
$ kubectl -n argo-events get workflows.argoproj.io
webhook-l7zsp   Succeeded   5m
{{% / highlight %}}

The below images are from the Agro Workflows UI.

# Summary
Technically Argo Events and Argo Workflows do not integrate with each other so it would be more accurtate to say that they work well together. Argo Events and Argo Workflows are two independant products, each addressing their specific use cases.

The process of getting Argo Events and Argo Workflows working together wasn't that difficult, my biggest stumbling points were RBAC and understanding how the components worked together.

# Appendix
The appendix provides documentation links for further reading.

## Agro Events
* [Architecture](https://argoproj.github.io/argo-events/concepts/architecture/)
* [Service Accounts](https://argoproj.github.io/argo-events/service-accounts/#service-account-for-sensors)