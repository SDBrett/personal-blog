---
author: Brett Johnson
postseries: operators
categories:
- Kubernetes
- Operators
date: "2021-02-08"
image: /assets/images/operator_logo_framework_color.svg
tags:
- Kubernetes
- Operators
title: Operator SDK Part 1
versions:
- software: Kubernetes
  versions:
  - v1.17.x
  - v1.18.x
  - v1.19.x
draft: false
---

The Operator-SDK is a command-line tool used by operator developers to build, test and package their operators. The SDK assists developers transform application business logic into a package which extends the Kubernetes API.

## Initializing a new project

You need to provide the SDK with some necessary information to initialize the scaffolding for a new project. The initialization command is; `operator-SDK init --domain $DOMAIN --plugin $PLUGIN`.

**Parameters**

`--domain` sets the domain for API resources that this operator will create on the cluster.

`--plugin` specifies the scaffolding structure and type of operator being build. If a plugin is not specified, the value defaults to Go. You can see the different scaffolding structures [here](#scaffolding).

## Plugins

The SDK can build operators based on one of three technologies; Ansible, Helm, and Go. The choice of plugin affects how to write the business logic and your operators capabilities and limitations.

At the time of writing Helm can only achieve level 2 maturity level where Ansible and Go can achieve a maturity level of 5. The below image is accurate at the time of writing.

![Plugin Maturity](/assets/images/plugin-maturity.png)

Image source: https://operatorframework.io/operator-capabilities/

**Ansible**

Business logic is written operator with Ansible playbooks; these playbooks are mapped to Kubernetes resources using the `watches.yaml` file. The mapping designates what playbooks to execute by resource type and other configured conditions. 

There are two primary packages inside an Ansible operators container, the Ansible Operator Binary and the Ansible Runner.

The Ansible Operator Binary gets resources from the Kubernetes API service, manages the resource status and triggers the Ansible Runner.

The Ansible Runner executes the Ansible playbook and sends events to the controller about the running process.

**Helm**
A Helm based operator consumes a one or more helm charts to manage an application. You can create charts specifically for the operator or import existing charts. The operator obtains values for charts from the `.spec` field of custom resources. 

The `watches.yaml` file creates mappings between Kubernetes resources and charts. Override values are also configured using `watches.yaml`; these allow you to override chart default values and supersede custom resource `.spec` values.

Override values can be configured as a static value or reference an environment variable from the operator pod. Environmental variables are configured on the manager pod by editing `config/manager/manager.yaml` and add flexibility when deploying the operator in different environments.

**Go**

A Go-based operator provides the highest level of flexibility but is the most complex to learn. The Go-based operator does not place an additional level of abstraction on top of the SDK as the Ansible and Helm operators do.

To write a custom resource, you first define the structure in a `_types.go` located in `api/$API_VERSION/` then run `make generate` to generate a `deep copy.go` file for each CRD. The deep file ensures that the new APIs (CRDs) implement the `runtime.Object` interface. 

The next step is to run `make manifests` to generate manifests from code. 

The controller and reconciliation loops still need to have application logic added before the operator is functional.

## Creating APIs

Operators extend the Kubernetes API by adding customer resources to the cluster. The command `operator-SDK create API` adds a new CRD and updates metadata within the operator project. 

Creating a CRD for either an Ansible or Helm based operator is done by writing the schema to the CRD manifest. For a Go-based operator, you defined the CRDs schema using Go code and generate the manifest.

The schema is defined in the manifest using the `openAPIV3Schema` specification. Specifying every data field and validation criteria in your CRD improves stability and security, but it can take a long time. For example, working on a Helm chart, you would need to transpose all the chart inputs into your CRD and then keep it aligned with future chart releases.

The field `x-Kubernetes-preserve-unknown-fields: true` can be added to a CRD to preserve fields which not in the schema. If this field is omitted or set to false, fields not in the schema are removed. You add this field directly to the manifest for Ansible and Helm based operators, but for a based operator, you add `// +kubebuilder:validation:XPreserveUnknownFields` to the field.

It is important to know the minimum Kubernetes version that you intend for your operator to be used on. There is a slightly different CRD structure between `apiextensions.k8s.io/v1beta1` and `apiextensions.k8s.io/v1`. `apiextensions.k8s.io/v1beta1` is deprecated and will be removed from Kubernetes in version 1.19; `apiextensions.k8s.io/v1` requires Kubernetes 1.16 or later.

Below is an example of a custom resource written for an Ansible based operator.

{{< highlight yaml >}}
---
apiVersion: apiextensions.k8s.io/v1beta1
kind: CustomResourceDefinition
metadata:
  name: vaults.vault.sdbrett.com
spec:
  group: vault.sdbrett.com
  names:
    kind: Vault
    listKind: VaultList
    plural: vaults
    singular: vault
  scope: Namespaced
  version: v1alpha1
  validation: 
    openAPIV3Schema:
      description: Vault is the Schema for the vaults API
      properties:
        apiVersion:
          description: 'APIVersion defines the versioned schema of this representation
            of an object. Servers should convert recognized schemas to the latest
            internal value, and may reject unrecognized values. More info: https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#resources'
          type: string
        kind:
          description: 'Kind is a string value representing the REST resource this
            object represents. Servers may infer this from the endpoint the client
            submits requests to. Cannot be updated. In CamelCase. More info: https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#types-kinds'
          type: string
        metadata:
          type: object
        spec:
          description: Spec defines the desired state of Vault
          type: object
          x-kubernetes-preserve-unknown-fields: true
          properties:
            targetNamespace:
              type: string
        status:
          description: Status defines the observed state of Vault
          type: object
          x-kubernetes-preserve-unknown-fields: true
      type: object
  served: true
  storage: true
  subresources:
    status: {}
{{< /highlight >}}

The above produces the follow CRD manifest.

{{< highlight yaml >}}
apiVersion: apiextensions.k8s.io/v1beta1
kind: CustomResourceDefinition
metadata:
  name: vaults.vault.sdbrett.com
spec:
  group: vault.sdbrett.com
  names:
    kind: Vault
    listKind: VaultList
    plural: vaults
    singular: vault
  scope: Namespaced
  served: true
  storage: true
  subresources:
    status: {}
  validation:
    openAPIV3Schema:
      description: Vault is the Schema for the vaults API
      properties:
        apiVersion:
          description: 'APIVersion defines the versioned schema of this representation of an object. Servers should convert recognized schemas to the latest internal value, and may reject unrecognized values. More info: https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#resources'
          type: string
        kind:
          description: 'Kind is a string value representing the REST resource this object represents. Servers may infer this from the endpoint the client submits requests to. Cannot be updated. In CamelCase. More info: https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#types-kinds'
          type: string
        metadata:
          type: object
        spec:
          description: Spec defines the desired state of Vault
          properties:
            targetNamespace:
              type: string
          type: object
          x-kubernetes-preserve-unknown-fields: true
        status:
          description: Status defines the observed state of Vault
          type: object
          x-kubernetes-preserve-unknown-fields: true
      type: object
  version: v1alpha1
{{< /highlight >}}

To write the same CRD for a Go-based operator, you would write the following:

{{< highlight go >}}
/*
Copyright 2021.

Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at

    http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.
*/

package v1alpha1

import (
  metav1 "k8s.io/apimachinery/pkg/apis/meta/v1"
)

// EDIT THIS FILE!  THIS IS SCAFFOLDING FOR YOU TO OWN!
// NOTE: JSON tags are required.  Any new fields you add must have json tags for the fields to be serialized.

// VaultsSpec defines the desired state of Vaults
type VaultsSpec struct {
  // INSERT ADDITIONAL SPEC FIELDS - desired state of cluster
  // Important: Run "make" to regenerate code after modifying this file

  TargetNamespace string `json:"targetNamespace,omitempty"`
}

// VaultsStatus defines the observed state of Vaults
type VaultsStatus struct {
  // INSERT ADDITIONAL STATUS FIELD - define observed state of cluster
  // Important: Run "make" to regenerate code after modifying this file
}

// +kubebuilder:object:root=true
// +kubebuilder:subresource:status

// Vaults is the Schema for the vaults API
type Vaults struct {
  metav1.TypeMeta   `json:",inline"`
  metav1.ObjectMeta `json:"metadata,omitempty"`

  Spec   VaultsSpec   `json:"spec,omitempty"`
  Status VaultsStatus `json:"status,omitempty"`
}

// +kubebuilder:object:root=true

// VaultsList contains a list of Vaults
type VaultsList struct {
  metav1.TypeMeta `json:",inline"`
  metav1.ListMeta `json:"metadata,omitempty"`
  Items           []Vaults `json:"items"`
}

func init() {
  SchemeBuilder.Register(&Vaults{}, &VaultsList{})
}
{{< /highlight >}}

The above code generates the following CRD manifest.

{{< highlight yaml >}}


---
apiVersion: apiextensions.k8s.io/v1
kind: CustomResourceDefinition
metadata:
  annotations:
    controller-gen.kubebuilder.io/version: v0.4.1
  creationTimestamp: null
  name: vaults.vault.sdbrett.com
spec:
  group: vault.sdbrett.com
  names:
    kind: Vaults
    listKind: VaultsList
    plural: vaults
    singular: vaults
  scope: Namespaced
  versions:
  - name: v1alpha1
    schema:
      openAPIV3Schema:
        description: Vaults is the Schema for the vaults API
        properties:
          apiVersion:
            description: 'APIVersion defines the versioned schema of this representation
              of an object. Servers should convert recognized schemas to the latest
              internal value, and may reject unrecognized values. More info: https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#resources'
            type: string
          kind:
            description: 'Kind is a string value representing the REST resource this
              object represents. Servers may infer this from the endpoint the client
              submits requests to. Cannot be updated. In CamelCase. More info: https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#types-kinds'
            type: string
          metadata:
            type: object
          spec:
            description: VaultsSpec defines the desired state of Vaults
            properties:
              targetNamespace:
                type: string
            type: object
          status:
            description: VaultsStatus defines the observed state of Vaults
            type: object
        type: object
    served: true
    storage: true
    subresources:
      status: {}
status:
  acceptedNames:
    kind: ""
    plural: ""
  conditions: []
  storedVersions: []
{{< /highlight >}}



### Scaffolding
Scaffolding is a projects directory and file structure, the structure different based on the plugin being used to write the operator.

The tables below describe the basic scaffolding for each operator plugin.

**Ansible**
| File/Folders   | Purpose |
| :---           | :---    |
| Dockerfile | The Dockerfile for building the container image for the operator. |
| Makefile | Contains make targets for building, publishing, deploying the container image that wraps the operator binary, and make targets for installing and uninstalling the CRD. |
| PROJECT | A YAML file containing meta information for the operator. |
| config/crd | The base CRD files and the customization settings. |
| config/default | Collects all operator manifests for deployment, used by `make deploy`. |
| config/manager | The controller manager deployment. |
| config/Prometheus | The ServiceMonitor resource for monitoring the operator. |
| config/RBAC | The role, role binding for leader election and authentication proxy. |
| config/samples | The sample resources created for the CRDs. |
| config/testing | Some sample configurations for testing. |
| playbooks/ | A subdirectory for the playbooks to run. |
| roles/ | A subdirectory for the roles tree to run. |
| watches.yaml | The Group, Version, and Kind of the resources to watch, and the Ansible invocation method. New entries are added via the 'create api' command. |
| requirements.yml | A YAML file containing the Ansible collections and role dependencies to install during build. |
| molecule/ | The [Molecule](https://molecule.readthedocs.io/) scenarios for end-to-end testing of your role and operator |

[Source](https://github.com/operator-framework/operator-sdk/tree/master/website/content/en/docs/building-operators/ansible/reference)

**Helm**

| File/Folders | Purpose                                                                           |
| :----------- | :-------------------------------------------------------------------------------- |
| config       | Contains kustomize manifests for deploying this operator on a Kubernetes cluster. |
| helm-charts/ | Contains a Helm chart initialized with `operator-SDK create api`.                 |
| Dockerfile   | Used to build the operator image with `make docker-build`.                        |
| watches.yaml | Contains Group, Version, Kind, and Helm chart location.                           |
| Makefile     | Contains the targets used to manage the project.                                  |
| PROJECT      | Contains meta-information about the project.                                      |

[Source](https://github.com/operator-framework/operator-sdk/blob/master/website/content/en/docs/building-operators/helm/reference/project_layout.md)

**Go**
| File/Folders   | Purpose |
| :---           | :---    |
| Dockerfile | The Dockerfile for building the container image for the operator. |
| Makefile | Contains make targets for building, publishing, deploying the container image that wraps the operator binary, and make targets for installing and uninstalling the CRD. |
| PROJECT | A YAML file containing meta information for the operator. |
| apis/$GROUP/$VERSION | APIs for multi API projects |
| config/crd | The base CRD files and the customization settings. |
| config/default | Collects all operator manifests for deployment, used by `make deploy`. |
| config/manager | The controller manager deployment. |
| config/Prometheus | The ServiceMonitor resource for monitoring the operator. |
| config/RBAC | The role, role binding for leader election and authentication proxy. |
| config/samples | The sample resources created for the CRDs. |
| config/testing | Some sample configurations for testing. |
| controllers/$GROUP | controller for multi API projects |
| hack/ | Script directory |
| main.go | Operator entry point |

## RBAC
You need to configure RBAC so you operator can watch and manage resources on the cluster. For Ansible and Helm based operators, RBAC management is done by managing `Role` and `RoleBinding` or `ClusterRole` and `ClusterRoleBinding` manifests. For a Go base operator, RBAC markers are added to the `controller.go` file.

Permissions are automatically added for CRDs created using the `operator-SDK create api` command, but additional permissions need to be set manually.

This is the base `config/rbac/role.yaml` for an Ansible / Helm project

{{< highlight yaml >}}
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: manager-role
rules:
  ##
  ## Base operator rules
  ##
  - apiGroups:
      - ""
    resources:
      - secrets
      - pods
      - pods/exec
      - pods/log
    verbs:
      - create
      - delete
      - get
      - list
      - patch
      - update
      - watch
  - apiGroups:
      - apps
    resources:
      - deployments
      - daemonsets
      - replicasets
      - statefulsets
    verbs:
      - create
      - delete
      - get
      - list
      - patch
      - update
      - watch
  ##
  ## Rules for vault.sdbrett.com/v1alpha1, Kind: Vaults
  ##
  - apiGroups:
      - vault.sdbrett.com
    resources:
      - vaults
      - vaults/status
      - vaults/finalizers
    verbs:
      - create
      - delete
      - get
      - list
      - patch
      - update
      - watch
# +kubebuilder:scaffold:rules

{{< /highlight >}}

This generates the following ClusterRole

{{< highlight yaml >}}
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: ans-op-manager-role
rules:
- apiGroups:
  - ""
  resources:
  - secrets
  - pods
  - pods/exec
  - pods/log
  verbs:
  - create
  - delete
  - get
  - list
  - patch
  - update
  - watch
- apiGroups:
  - apps
  resources:
  - deployments
  - daemonsets
  - replicasets
  - statefulsets
  verbs:
  - create
  - delete
  - get
  - list
  - patch
  - update
  - watch
- apiGroups:
  - vault.sdbrett.com
  resources:
  - vaults
  - vaults/status
  - vaults/finalizers
  verbs:
  - create
  - delete
  - get
  - list
  - patch
  - update
  - watch
{{< /highlight >}}

In Go, the ClusterRole is written to the `controllers/*_controller.go` file for each controller.

{{< highlight Go >}}
/*
Copyright 2021.

Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at

    http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.
*/

package controllers

import (
  "context"

  "github.com/go-logr/logr"
  "k8s.io/apimachinery/pkg/runtime"
  ctrl "sigs.k8s.io/controller-runtime"
  "sigs.k8s.io/controller-runtime/pkg/client"

  vaultv1alpha1 "github.com/sdbrett/go-op/api/v1alpha1"
)

// VaultsReconciler reconciles a Vaults object
type VaultsReconciler struct {
  client.Client
  Log    logr.Logger
  Scheme *runtime.Scheme
}

// +kubebuilder:rbac:groups=vault.sdbrett.com,resources=vaults,verbs=get;list;watch;create;update;patch;delete
// +kubebuilder:rbac:groups=vault.sdbrett.com,resources=vaults/status,verbs=get;update;patch
// +kubebuilder:rbac:groups=vault.sdbrett.com,resources=vaults/finalizers,verbs=update

// Reconcile is part of the main Kubernetes reconciliation loop which aims to
// move the current state of the cluster closer to the desired state.
// TODO(user): Modify the Reconcile function to compare the state specified by
// the Vaults object against the actual cluster state, and then
// perform operations to make the cluster state reflect the state specified by
// the user.
//
// For more details, check Reconcile and its Result here:
// - https://pkg.go.dev/sigs.k8s.io/controller-runtime@v0.7.0/pkg/reconcile
func (r *VaultsReconciler) Reconcile(ctx context.Context, req ctrl.Request) (ctrl.Result, error) {
  _ = r.Log.WithValues("vaults", req.NamespacedName)

  // your logic here

  return ctrl.Result{}, nil
}

// SetupWithManager sets up the controller with the Manager.
func (r *VaultsReconciler) SetupWithManager(mgr ctrl.Manager) error {
  return ctrl.NewControllerManagedBy(mgr).
    For(&vaultv1alpha1.Vaults{}).
    Complete(r)
}
{{< /highlight >}}

This code generates the following ClusterRole manifest.

{{< highlight yaml >}}

---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  creationTimestamp: null
  name: manager-role
rules:
- apiGroups:
  - vault.sdbrett.com
  resources:
  - vaults
  verbs:
  - create
  - delete
  - get
  - list
  - patch
  - update
  - watch
- apiGroups:
  - vault.sdbrett.com
  resources:
  - vaults/finalizers
  verbs:
  - update
- apiGroups:
  - vault.sdbrett.com
  resources:
  - vaults/status
  verbs:
  - get
  - patch
  - update
  
{{< /highlight >}}

## Summary
So far, we have looked at how the operator-SDK is to build operators based on Ansible, Helm and Go. We've also seen some of the difference between building operators based on different technologies.

The next post looks at using the SDK to build, push and run your operator.
