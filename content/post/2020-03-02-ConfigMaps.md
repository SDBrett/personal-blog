---
author: Brett Johnson
categories:
- Kubernetes
- Containers
date: "2020-03-02T00:00:00Z"
image: /assets/images/Kubernetes_logo.svg
tags:
- Kubernetes
title: Kubernetes Configmaps
versions: null
---

ConfigMaps enable decoupling of application configuration settings and a pod, improving workload portability. ConfigMaps are a key/value store for non-sensitive information such as; command line argument values, environment-specific strings and URL's. 

Do not use ConfigMaps to store sensitive or encrypted data; instead, use secrets for this use case.

You can create a ConfigMap by using the command `kubectl create configmap [NAME] [DATA]`, where data is entered as literal values or read from a file.

{{< highlight shell >}}

# Create ConfigMap with literal values
~/:$ kubectl create configmap literaltest --from-literal=key1=value1 --from-literal=key2=value2
~/:$ kubectl get configmap literaltest -o yaml
apiVersion: v1
data:
  key1: value1
  key2: value2
kind: ConfigMap
metadata:
  creationTimestamp: 2020-01-16T20:22:25Z
  name: literaltest
  namespace: default
  resourceVersion: "409447"
  selfLink: /api/v1/namespaces/default/configmaps/literaltest
  uid: 653365ef-bf5c-4ef3-a88e-069c4fe0f144

# Create ConfigMap with data from specific files within a directory
~/:$ kubectl create configmap filetest --from-file=configTest/configs/file1.yml --from-file=configTest/configs/file2.json
~/:$ kubectl get configmap filetest -o yaml
apiVersion: v1
data:
  file1.yml: "key1: value1\r\nkey2: value2"
  file2.json: "{\r\n  \"keys\": {\r\n  \"key3\": [\"value3.1\",\"value3.2\"],\r\n
    \ \"key4\": \"value4\"\r\n  }\r\n}"
kind: ConfigMap
metadata:
  creationTimestamp: 2020-01-16T20:25:15Z
  name: filetest
  namespace: default
  resourceVersion: "409674"
  selfLink: /api/v1/namespaces/default/configmaps/filetest
  uid: ee5cda44-03f8-4b37-a6e4-2045375f9d87

# Create ConfigMap with data from all files within a directory
~/:$ kubectl create configmap dirtest --from-file=configTest/configs/ 
~/:$ kubectl get configmap dirtest -o yaml
apiVersion: v1
data:
  file1.yml: "key1: value1\r\nkey2: value2"
  file2.json: "{\r\n  \"keys\": {\r\n  \"key3\": [\"value3.1\",\"value3.2\"],\r\n
    \ \"key4\": \"value4\"\r\n  }\r\n}"
  file3.cnf: "key5: value5\r\nkey6: value6"
kind: ConfigMap
metadata:
  creationTimestamp: 2020-01-16T20:26:11Z
  name: dirtest
  namespace: default
  resourceVersion: "409746"
  selfLink: /api/v1/namespaces/default/configmaps/dirtest
  uid: 754e4806-c4e2-41e4-bea2-b768469ef711

{{< / highlight >}}

# ConfigMap data in a pod

The following examples demonstrate a Pod consuming ConfigMaps as VolumeMounts and environmental variables.

## VolumeMounts

Pods can mount ConfigMaps as volumes, populating a directory with the ConfigMaps data. Files are created within the directory for each ConfigMap key; each file contains the data from the relative key.

Let's create a pod that consumes our ConfigMaps using VolumeMounts.

{{< highlight yaml >}}
apiVersion: v1
kind: Pod
metadata:
  name: cm-vol-pod
spec:
  containers:
  - name: cm-vol-pod
    image: nginx
    volumeMounts:
    - name: literaltest
      mountPath: "/etc/literaltest"
    - name: filetest
      mountPath: "/etc/filetest"
    - name: dirtest
      mountPath: "/etc/dirtest"
  volumes:
  - name: literaltest
    configMap: 
      name: literaltest
  - name: filetest
    configMap: 
      name: filetest
  - name: dirtest
    configMap: 
      name: dirtest
      items:
      - key: file3.cnf

{{< / highlight >}}

**Create pod and view mounts**

{{< highlight shell >}}
~/:$ kubectl apply -f .\configtest\configs\cm-pod-vm.yml
~/:$ kubectl exec -it cm-vol-pod /bin/sh

root@cm-vol-pod:/# ls -la /etc/
# Edited for berevity
drwxr-xr-x 1 root root    4096 Jan 29 20:11 .
drwxr-xr-x 1 root root    4096 Jan 29 20:11 ..
drwxrwxrwx 3 root root    4096 Jan 29 20:11 dirtest
drwxrwxrwx 3 root root    4096 Jan 29 20:11 filetest
drwxrwxrwx 3 root root    4096 Jan 29 20:11 literaltest

root@cm-vol-pod:/# ls -la /etc/dirtest/ /etc/filetest/ /etc/literaltest/
/etc/dirtest/:
total 12
drwxrwxrwx 3 root root 4096 Jan 29 20:11 .
drwxr-xr-x 1 root root 4096 Jan 29 20:11 ..
drwxr-xr-x 2 root root 4096 Jan 29 20:11 ..2020_01_29_20_11_08.107144640
lrwxrwxrwx 1 root root   31 Jan 29 20:11 ..data -> ..2020_01_29_20_11_08.107144640
lrwxrwxrwx 1 root root   16 Jan 29 20:11 file3.cnf -> ..data/file3.cnf

/etc/filetest/:
total 12
drwxrwxrwx 3 root root 4096 Jan 29 20:11 .
drwxr-xr-x 1 root root 4096 Jan 29 20:11 ..
drwxr-xr-x 2 root root 4096 Jan 29 20:11 ..2020_01_29_20_11_08.531974309
lrwxrwxrwx 1 root root   31 Jan 29 20:11 ..data -> ..2020_01_29_20_11_08.531974309
lrwxrwxrwx 1 root root   16 Jan 29 20:11 file1.yml -> ..data/file1.yml
lrwxrwxrwx 1 root root   17 Jan 29 20:11 file2.json -> ..data/file2.json

/etc/literaltest/:
total 12
drwxrwxrwx 3 root root 4096 Jan 29 20:11 .
drwxr-xr-x 1 root root 4096 Jan 29 20:11 ..
drwxr-xr-x 2 root root 4096 Jan 29 20:11 ..2020_01_29_20_11_08.723801119
lrwxrwxrwx 1 root root   31 Jan 29 20:11 ..data -> ..2020_01_29_20_11_08.723801119
lrwxrwxrwx 1 root root   11 Jan 29 20:11 key1 -> ..data/key1
lrwxrwxrwx 1 root root   11 Jan 29 20:11 key2 -> ..data/key2

root@cm-vol-pod:/# cat /etc/dirtest/file1.yml
key1: value1
key2: value2# cat /etc/dirtest/file2.json
{
  "keys": {
  "key3": ["value3.1","value3.2"],
  "key4": "value4"
  }
}
{{< / highlight >}}

The ConfigMaps are mounted as directories in the locations specified in the manifest. We can see that the key names are used as filenames, and files contain `ConfigMap.[KEY].data`. 

ConfigMap VolumeMaps follow the same 'rules' as normal VolumeMounts.
- Each `mountPath` value must be unique. You cannot mount two ConfigMaps to the same directory
- The `mountPath` directory will only contain files from the ConfigMaps. Example below.

{{< highlight shell >}}
root@cm-vol-pod:/# ls /etc/fonts
conf.avail  conf.d  fonts.conf

root@cm-vol-pod:/# exit

~/:$ kubectl delete pod cm-vol-pod

# Changed literaltest.mountPath to /etc/fonts
~/:$ kubectl apply -f .\configtest\configs\cm-vol-pod.yml
~/:$ kubectl exec -it cm-vol-pod /bin/sh
root@cm-vol-pod:/# ls /etc/fonts
key1  key2
{{< / highlight >}}

The kubelet is responsible for synchronizing changes to the ConfigMap object with the pods consuming the ConfigMap as a VolumeMount. Any updates made to a ConfigMap will eventually be reflected in any pods consuming that ConfigMap as a VolumeMount.

{{< highlight shell >}}
~/:$ kubectl exec -it cm-vol-pod /bin/sh
root@cm-vol-pod:/# cat /etc/literaltest/key1
value1

# Update ConfigMap
~/:$ kubectl edit cm literaltest
~/:$ kubectl get cm literaltest -o yaml
apiVersion: v1
data:
  key1: updated
  key2: value2
kind: ConfigMap
metadata:
  creationTimestamp: 2020-01-16T20:22:25Z
  name: literaltest
  namespace: default
  resourceVersion: "1160455"
  selfLink: /api/v1/namespaces/default/configmaps/literaltest
  uid: 653365ef-bf5c-4ef3-a88e-069c4fe0f144

# Check file data
~/:$ kubectl exec -it cm-vol-pod /bin/sh
root@cm-vol-pod:/# cat /etc/literaltest/key1
updated
{{< / highlight >}}


## Consuming as environmental variables

Container environmental variables can be defined using data from ConfigMaps.

ConfigMaps previously created will be used for following examples.

{{< highlight yaml >}}
apiVersion: v1
kind: Pod
metadata:
  name: cm-env-pod
spec:
  containers:
  - name: cm-env-pod
    image: nginx
    env:
      - name: LITERALTEST
        valueFrom: 
          configMapKeyRef: 
            name: literaltest
            key: key1
      - name: LITERALTEST2
        valueFrom: 
          configMapKeyRef: 
            name: literaltest
            key: key2
      - name: FILETEST
        valueFrom: 
          configMapKeyRef:
            name: filetest
            key: file1.yml
      - name: FILETEST2
        valueFrom: 
          configMapKeyRef:
            name: filetest
            key: file2.json

{{< / highlight >}}

{{< highlight shell >}}
~/:$ kubectl apply -f .\configtest\configs\cm-env-pod.yml
~/:$ kubectl exec -it cm-env-pod /bin/sh

~/:$ kubectl exec -it cm-env-pod /bin/sh
env echo $LITERALTEST
value1
root@cm-env-pod:/# echo $LITERALTEST2
value2
root@cm-env-pod:/# echo $FILETEST
key2: value2
root@cm-env-pod:/# echo $FILETEST2
 }key4": "value4".1","value3.2"],
{{< / highlight >}}

The environment variables that contain data the string value for the specified key. The data stored in the environmental variables which have data from the filetest ConfigMap not the same as the file content; but, FILETEST only contains the second line, and FILETEST2 is very different from the original file. ConfigMaps containing data that originated from files are not appropriate for use as environmental variables.

ConfigMap defined environmental variables can be used as variables in pod commands.

{{< highlight yaml >}}
apiVersion: v1
kind: Pod
metadata:
  name: cm-env-pod2
spec:
  containers:
    - name: cm-env-pod2
      image: nginx
      command: [ "/bin/sh", "-c", "echo $(LITERALTEST)"]
      env:
        - name: LITERALTEST
          valueFrom: 
            configMapKeyRef: 
              name: literaltest
              key: key1
  restartPolicy: Never
{{< / highlight >}}

{{< highlight shell >}}
~/:$ kubectl apply -f .\configtest\configs\cm-env-pod2.yml
value1
{{< / highlight >}}

Unlike when using VolumeMounts, the kubelet does not synchronise ConfigMap data with containers when ConfigMaps consumed as environmental variables.

{{< highlight shell >}}
~/:$ kubectl exec -it cm-env-pod /bin/sh
root@cm-env-pod:/# echo $LITERALTEST
value1

# Update ConfigMap
~/:$ kubectl edit cm literaltest
~/:$ kubectl get cm literaltest -o yaml
apiVersion: v1
data:
  key1: updated
  key2: value2
kind: ConfigMap
metadata:
  creationTimestamp: 2020-01-16T20:22:25Z
  name: literaltest
  namespace: default
  resourceVersion: "1162145"
  selfLink: /api/v1/namespaces/default/configmaps/literaltest
  uid: 653365ef-bf5c-4ef3-a88e-069c4fe0f144

# Check environmental variable data
~/:$ kubectl exec -it cm-env-pod /bin/sh
root@cm-env-pod:/# echo $LITERALTEST
value1
{{< / highlight >}}

## Summary

ConfigMaps are a frequently used object within Kubernetes environments due to their decoupling functionality and application portability improvements.