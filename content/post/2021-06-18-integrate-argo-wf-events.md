---
author: Brett Johnson
categories:
- Kubernetes
- Argo
date: "2021-06-18"
image: /assets/images/argo-horizontal-color.png
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

![Argo Logo](/assets/images/argo-horizontal-color.png)

This post demonstrates how to use Argo Events and Argo Workflows to achieve event driven workflow execution. Event based architectures are a key part of building solutions where the individual components are decoupled, the line of responsibility for the action which created an event stops at point of generating the event, what happens next is a problem for another system.

# Core Components

This section provides a quick overview of the core components for Argo Events and Argo Workflows. All of these components are Kubernetes resources which are consumed to deploy and configure additional resources on the cluster such as deployments.

## Argo Events

![Argo Event Process](/assets/images/argo-events-event-process.svg)

**Event Source** 
The `eventsources.argoproj.io` resource specifies how to consume events from external services such as; GitHub, Slack and Webhooks. The resource also specifies criteria to compare observed events against.

The Event Source process runs within a pod managed by the `eventsource-controller`. The process writes to the eventbus when it observes events which match the filtering criteria.

A service account with `[List, Watch]` permissions is required for the Event Source to monitor Kubernetes resources.

**Event Bus**
An Event Bus is a transport service for events to go from an Event Source to a Sensor. The Event Bus process runs in a cluster of three pods managed by the `eventbus-controller`.

**Sensors**
A `sensors.argoproj.io` resource specifies the events to look for on the Event Bus and the response to trigger when a matching event is observed.

The sensor process runs in a pod managed by the `sensor-controller`.

There are currently 10 predefined triggers for services such as; HTTP requests, Kubernetes Objects and Slack. Additionally, you can create your own triggers as needed.

A service account with sufficient permissions is required if the trigger manipulates Kubernetes Objects.

## Argo Workflows


**Workflows**
A `workflows/argoproj.io/v1alpha1` resource defines a task to be executed by a workflow and the intent to execute. Workflow execution is immediate and a once off event. Use the  `cronWorkflows/argoproj.io/v1alpha1` resource to create workflows with delayed execution or execute on a recurring schedule.

**Workflow Controller**

The Workflow Controller watches the Kubernetes API server for `workflows/argoproj.io/v1alpha1` and `cronWorkflows/argoproj.io/v1alpha1` resources. You specify the controllers scope to either it's own namespace or the entire cluster using the Argo Workflows deployment manifest.  A cluster wide scope is required for this integration to work as the Workflow Controller needs to observe the creation of `workflows/argoproj.io/v1alpha1` resources in the argo-events namespace.

**Executor Pod**
A workflows process runs inside a pod called the `executor pod` which is created by the Workflow Controller. The executor pod authenticates against with the Kubernetes API server using the service account specified in the pods configuration. The `default` service account of the namespace which the executor pod is running in is used if the workflows `spec.serviceAccountName` field is empty.

# Integration Example

Our use case for this example is to allow uses to deploy a workload based from an external system. The external system provides the users with a request form, once submitted the system sends a HTTP request causing deployment of the workload.

Our task is to deploy a solution which deploys the workflow when the HTTP request is received.


The diagram below is the user level view of our example workflow, they submit a request and a workload is deployed.

![User view](/assets/images/argo-integrate-event-wf-user-view.svg)

This diagram details the data and process flow or our example workflow. 

![Process view](/assets/images/argo-integrate-event-wf-process-view.svg)

## Deployment Topology
The below diagram is a high level diagram showing where the Argo Event and Argo Workflow service processes run at a namespace level. This diagram can be built on top of to show more detailed and specific information.

![Topology view](/assets/images/argo-integrate-event-wf-topology.svg)


The diagram also tells us that the Workflow Controller must be able to read `workflows/argoproj.io/v1alpha1` resources from other namespaces, therefore we must deploy Argo  Workflows with a cluster wide scope.

## Argo Workflow Deployment

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

## Argo Events Deployment

The commands below deploy Argo Events controllers and the Event Bus service.

{{% highlight shell %}}
$ kubectl create ns argo-events
# Install Argo Events
$ kubectl apply -f https://raw.githubusercontent.com/argoproj/argo-events/stable/manifests/install.yaml

# Deploy the Event Bus
$ kubectl apply -n argo-events -f https://raw.githubusercontent.com/argoproj/argo-events/stable/examples/eventbus/native.yaml
{{% / highlight %}}

**Event Source**
Our workflow needs to be able to receive HTTP Requests from the external system, this is achieved using an WebHook EventSource. [Manifest Source](https://raw.githubusercontent.com/SDBrett/argocd-examples/main/events/event-source.yaml)

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
* Create a service which forwards port 12000 to the Event Source pod on port 12000

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
# Validate writing to Event Bus
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
Our integration example manipulates Kubernetes resources, therefore we need to create service accounts with the correct access rights to succeed. We need to understand how Argo Events and Argo Workflows behaves in relation to our use case.

### Sensor Service Account
The sensor process runs within a pod in the `argo-events` namespace and the trigger creates a `workflows/argoproj.io/v1alpha1` resource. The resource spec in the workflow resource does not specify which namespace to create the resource in, therefore it will be created in the `argo-events` namespace.

The sensor requires a service account with permission to create `workflows/argoproj.io/v1alpha1` resources in the `argo-events` namespace. [Manifest Source](https://raw.githubusercontent.com/SDBrett/argocd-examples/main/events/sensor-service-accounts.yaml)

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

{{% highlight shell %}}
$ kubectl -n argo-events apply -f sensor-service-accounts.yaml
{{% / highlight %}}


### Workflow Service Account

The executor pod will be created in the `argo-events` namespace because that is where the `workflows/argoproj.io/v1alpha1` resource resides. 

The workflow process within the executor pod requires permissions to create a pod (the example workload) in the `argo-events` namespace

Below is the manifest for the service account used by the executor pod and the role and role bindings. [Manifest Source](https://raw.githubusercontent.com/SDBrett/argocd-examples/main/workflow/first-workflow-service-account.yaml)

{{% highlight yaml %}}
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
{{% / highlight %}}

{{% highlight shell %}}
$ kubectl -n argo-events apply -f first-workflow-service-account.yaml
{{% / highlight %}}


## Sensor Manifest

The manifest below specifies the configuration for our Sensor. [Manifest source](https://raw.githubusercontent.com/SDBrett/argocd-examples/main/events/webhook-sensor.yaml)

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
$ kubectl -n argo-events apply -f webhook-sensor.yaml
# The sensor controller creates a deployment for the sensor
$ kubectl  -n argo-events get deployments -l sensor-name=webhook
NAME                   READY   UP-TO-DATE   AVAILABLE   AGE
webhook-sensor-2hgxx   1/1     1            1           12s
{{% / highlight %}}

# End to end testing

All the pieces should be in place so our users can execute a workflow from an external system. Each item has been tested individually and we now need to validate the end to end process by sending a HTTP Request to the event source using curl.

{{% highlight shell %}}
$ curl -d '{"message":"Dont call me Shirley"}' -H "Content-Type: application/json" -X POST http://localhost:12000/deploy
$ kubectl -n argo-events get workflows.argoproj.io
NAME            STATUS    AGE
webhook-l7zsp   Running   8s 
$ kubectl -n argo-events get workflows.argoproj.io
webhook-l7zsp   Succeeded   5m
{{% / highlight %}}

The below images are from the Argo Workflows UI.

![Test Result 1](/assets/images/argo-integration-test-result1.png)
![Test Result 2](/assets/images/argo-integration-test-result2.png)

# Update 21/09/2021

In the example used for this post, the whalesay pod writes the entire Argo Events message to stdout. The message includes the HTTP request, though it is base64 encoded.

Natalie Caruna pointed has point out that to the Sensor manifest will cause the pod to display only the message content.

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
      eventName: deploy
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
                    args: ["{{inputs.parameters.bucket}}","{{inputs.parameters.key}}"]
          parameters:
            - src:
                dependencyName: test-dep
                dataKey: body.bucket
              dest: spec.arguments.parameters.0.value
            - src:
                dependencyName: test-dep
                dataKey: body.key
              dest: spec.arguments.parameters.1.value
{{% / highlight %}}


# Summary
Technically Argo Events and Argo Workflows do not integrate with each other so it would be more accurate to say that they work well together. Argo Events and Argo Workflows are two independent products, each addressing their specific use cases.

The process of getting Argo Events and Argo Workflows working together wasn't that difficult, my biggest stumbling points were RBAC and understanding how the components worked together.
