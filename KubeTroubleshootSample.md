---
sidebar_label: "Debugging Reference"
title: "Debug Options in Kubernetes"
description: "Useful commands, descriptions and links for debugging in Kubernetes"
icon: ""
hide_table_of_contents: false
sidebar_position: 10
tags: ["reference", "debug", "kubernetes", "k8s", "kube", "troubleshoot"]
---

# Debug Options in Kubernetes
This document covers different debugging options available in Kubernetes. Debugging can help improve performance, availability, and reliability in your Kubernetes environments, as well as identify vulnerabilities and errors. Regular audits and debugging can also help prevent future issues from occuring to begin with.
Kubernetes has [their debugging instructions located here](kubernetes.io/docs/tasks/debug/debug-cluster). 
This guide more briefly covers a greater array of debugging options in Kubernetes. These options are largely based around the `kubectl` command, as such, using these options requires using your CLI.

## List Cluster Resources
### `kubectl get`
The first step to debugging anything is knowing the state of items on your cluster. The best way to do this is to list the resources in your cluster. This is done with `kubectl get`. Some `get` examples are given below:

#### `kubectl get pods`
`kubectl get pods` lists all pods in your current namespace. 
```console
kubectl get pods
NAME    READY   STATUS  RESTARTS    AGE
pod1    1/1     Running 0           1m37s
pod2    1/1     Running 0           1m37s
```

Useful options with this command: 
* `-A`: Lists all pods in all namespaces
* `-n <namespace_name>`: Lists all pods in a specific namespace
* `-o wide`: Lists all pods in your current namespace with more information. More generally, `-o` changes the output format of your resource.

#### `kubectl get nodes`
`kubectl get nodes` lists all nodes in your cluster.
```console
kubectl get nodes
NAME    STATUS  ROLES   AGE     VERSION
node1   Ready   <none>  4m45s   v1.31.1
node2   Ready   <none>  4m45s   v1.31.1
```

Useful options with this command:
* `-o wide`: Lists all nodes with more information. More generally, `-o` changes the output format of your resource.

#### `kubectl get services`
`kubectl get services` lists all services in your current namespace.
```console
kubectl get services
NAMESPACE   NAME        TYPE        CLUSTER-IP  EXTERNAL-IP
default     Kubernetes  ClusterIP   10.0.0.1    <none>
NS1         test        ClusterIP   10.0.95.30  <none>    
```

Useful options with this command:
* `-A`: Lists all services in all namespaces

#### `kubectl get events`
`kubectl get events` lists all recent events for resources in your system.
```console
kubectl get events
LAST SEEN   TYPE    REASON      KIND    MESSAGE
12h         Normal  Pulling     Pod     Pulling image "images/testimage"
45m         Warning Failed      Pod     Error: ImagePullBackOff
47m         Normal  Starting    Node    Starting kubelet.
```
Useful options with this command: 
* `-A`: Lists all events in all namespaces
* `--sort-by=<criteria>`: Sorts your options listed by a given criteria. For example, timestamps in order to see your most recent events
* `-o wide`: Lists all events in your current namespace with more information. More generally, `-o` changes the output format of your resource.
* `--field-selector type=Warning`: Lists all Warning events.

Click [here](kubernetes.io/docs/reference/kubectl/generated/kubectl_get/) for the full documentation for `kubectl get`.

### `kubectl describe`
Another command used to get more information on your Kubernetes Cluster(s) is `kubectl describe`. This command gives details on a specific resource or group of resources. For instance, after using `kubectl get pods` you found that one of your pods was not running. You could next use `kubectl describe <pod-name>` to find more information on that specific pod. Some examples are given below:

#### `kubectl describe pod demo-pod`
```console
Name:               demo-pod
Namespace:          default
Priority:           0
Service Account:    default
Node:               Node1
Start Time:         Sun, 06 OCT 2024 17:22:52 -0500
Labels:             app-nginx
Annotations:        <none>
Status:             Running
IP:                 10.244.0.9
IPs:
    IP: 10.244.0.9
Containers:
    nginx:
        Container ID:   docker://0ad37590c12305238ed4f346cee1e40422c9e393d0a7bd477d8e23e4a112
        Image:          nginx:latest
        Image ID:       docker-pullable://nginx@sha256:87e44b7c739b204b0fd3a3418d74568c769dc8285b3d47c3a21bb86d3c3
        Port:           80/TCP
        Host Port:      0/TCP
        State:          Running
            Started:    Sun, 06 OCT 2024 17:22:52 -0500
        Ready:          True
        Restart Count:  0
        Environment:    <none>
        Mounts:
            /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-7chct (ro)
Conditions:
    Type                Status
    Initialized         True
    Ready               True
    ContainersReady     True
    PodScheduled        True
Volumes:
    kube-api-access-7chct:
        Type:                   Projected (a volume that contains injected data from multiple sources)
        TokenExpirationSeconds  3607
        ConfigMapName           kube-root-ca.crt
        ConfigMapOptional       <nil>
        Downward API:           true
QoS Class:                      BestEffort
Node-Selectors:                 <none>
Tolerations:                    node.kubernetes.io/notready:NoExecute op=Exists for 300s
                                node.kubernetes.io/unreachable:NoExecute op=Exists for 300s
Events:
    Type    Reason      Age     From                Message
    Normal  Scheduled   51s     default-scheduler   Successfully assigned default/demo-pod to mini-kube
    Normal  Pulling     51s     kubelet             Pulling image "nginx:latest"
    Normal  Pulled      48s     kubelet             Successfully pulled image "nginx:latest" in 2.750987244s (2.751889732s including waiting)
    Normal  Created     48s     kubelet             Created container nginx
    Normal  Started     48s     kubelet             Started container nginx
```
Useful options with this command: 
* `-A`: Describes the requested resources across all namespaces
* `-n <namespace_name>`: Describes the requested resource in a specific namespace

#### `kubectl describe node demo-node`
```console
kubectl describe node demo-node
Name:               demo-node
Roles:              admin
Labels:             beta.kubernetes.io/arch=amd64
                    beta.kubernetes.io/os=linux
                    kubernetes.io/arch=amd64
                    kubernetes.io/hostname=demo-node
                    kubernetes.io/os=linux
                    node-role.kubernetes.io/admin=
Annotations:        kubeadm.alpha.kubernetes.io/cri-socket: /var/run/dockerd.sock
                    node.alpha.kubernetes.io/ttl: 0
                    volumes.kubernetes.io/controller-managed-attach-detach: true
CreationTimeStamp:  Sun, 06 OCT 2024 17:21:51 -0500
Taints:             <none>
Unschedulable:      false
Conditions:
    Type            Status  LastHeartBeatTime               LastTransitionTime              Reason                      Message
    MemoryPressure  False   Sun, 06 OCT 2024 18:45:37 -0500 Sun, 06 OCT 2024 17:21:39 -0500 KubeletHasSufficientMemory  kubelet has sufficient memory available
    DiskPressure    False   Sun, 06 OCT 2024 18:45:37 -0500 Sun, 06 OCT 2024 17:21:39 -0500 KubeletHasNoDiskPressure    kubelet has no disk pressure
    PIDPressure     False   Sun, 06 OCT 2024 18:45:37 -0500 Sun, 06 OCT 2024 17:21:39 -0500 KubeletHasSufficientPID     kubelet has sufficient PID available
    Ready           True    Sun, 06 OCT 2024 18:45:37 -0500 Sun, 06 OCT 2024 17:21:39 -0500 KubeletReady                kubelet is posting ready status
```

Click [here](kubernetes.io/docs/reference/kubectl/generated/kubectl_describe/) for the full documentation for `kubectl describe`.

### `kubectl cluster-info dump`
A final command used to gain information about your cluster is `kubectl cluster-info dump`. This command outputs all relevant information regarding the clusters current state. This includes events, configurations, and logs. It is used to give a broad overview of your cluster's status.

Click [here](kubernetes.io/docs/reference/kubectl/generated/kubectl_cluster-info/kubectl_cluster-info_dump/) for the full documentation for `kubectl cluster-info dump`.

## Review your Cluster's Logs
Reviewing logs in your cluster is useful for finding errors while debugging. Logs will often give errors that will explain why a resource in your Kubernetes environment is non-operational. This is used to gain a deeper understanding of why a part of your cluster is non-operational. An example of `kubectl logs` is given below:
```console
kubectl logs --tail=10 demo-node
2024/10/06 17:58:08 [notice] 1#1: start worker process 41
2024/10/06 17:58:08 [notice] 1#1: start worker process 42
2024/10/06 17:58:08 [notice] 1#1: start worker process 43
2024/10/06 17:58:08 [notice] 1#1: start worker process 44
2024/10/06 17:58:08 [notice] 1#1: start worker process 45
2024/10/06 17:58:08 [notice] 1#1: start worker process 46
2024/10/06 17:58:08 [notice] 1#1: start worker process 47
2024/10/06 17:58:08 [notice] 1#1: start worker process 48
2024/10/06 17:58:08 [notice] 1#1: start worker process 49
2024/10/06 17:58:08 [notice] 1#1: start worker process 50
```
Useful options with this command: 
* `--since=1h`: Prints logs within the last hour
* `--tail=10`: Prints the last 10 lines of logs
* `-f`: Prints logs and follows any new logs
* `-c <container-name>`: Prints logs for a specific container in a pod
* `<file-name>.log`: Ending your command with a file name allows you to output your logs to that file for further refinement

Click [here](kubernetes.io/docs/reference/kubectl/generated/kubectl_logs/) for full documentation for `kubectl logs`.

## Test your Pod
`kubectl debug` creates an ephemeral copy of your target container(s) in a pod. This allows you to then test things in this container without interrupting or impacting your live container(s). Use this command to test a container which has errors or is non-operational. An example is given below: 
```console
kubectl debug -it node/node1 --image=nginx:latest
Creating debugging pod node-debugger-node1-bnm4e with container debugger on node node1
```
Useful options with this command: 
* `-i`: Keep stdin open on the container(s) in the pod
* `-t`: Allocate TTY for debugging the container
* `--image`: Image used to create your ephemeral container.

Click [here](kubernetes.io/docs/reference/kubectl/generated/kubectl_debug/) for the full documentation for `kubectl debug`.

## Issue CLI Commands Inside of your Containers
`kubectl exec` allows you to issue CLI commands from inside your containers. This is useful way to check configuration files, validate a volume mounting, or troubleshoot application behaviour. `kube exec` provides a direct line into your containers, and allows you to operate as you would in a traditional linux environment. Write the command you wish to issue after the `--` as shown below:
```console
kubectl exec -it pod1 --container nginxtest -- ls /usr/share/nginx/html
50x.html
index.html
```
Useful options with this command: 
* `-i`: Keep stdin open on the container(s) in the pod
* `-t`: Allocate TTY for debugging the container
* `--container`: The name of the container you are issuing your command in

<!--- 
# Logging and Monitoring Options
Though not directly debugging, logging and monitoring help identify issues in your Kubernetes environment by tracking key metrics. This can identify problems quickly and allows for automation and insight into ensuring your cluster's health and performance.

Kube Native Logging and Monitoring
Prometheus (Grafana?)
Datadog
Calico

Namespaces and Tagging?? Digests??

Other kube commands:
kubectl proxy - 
kubectl port-forward - forwards one or more local ports to a pod. useful for directly connecting to clusters without outside services/endpoints
netshoot
kubectl
kubectl explain?

-->

<!---
# Debug Options in Kubernetes

This document covers different debugging options available in Kubernetes. Debugging can help improve performance, availability, and reliability in your Kubernetes environments, as well as identify vulnerabilities and errors. Errors in your Kubernetes environment can sometimes occur due to errors with the applications running in those environments. Always ensure your application is running correctly.Regular audits and debugging can also help prevent future issues from occuring to begin with.

There are multiple layers on which issues can occur in Kubernetes: 
* The Application Layer, which directly hosts applications via containers
* The Object Layer, including pods and services
* The Infrastructure layer, which organizes nodes and networks
* The Control Plane layer

:::note

Both Beginner and Advance debugging options are covered in this document. 

:::

## Application Layer Debugging
?? Is this too basic or out of scope? direct container interaction? `kubectl exec` ideas?

## Object Layer Debugging
The Object Layer covers pods and services in Kubernetes. 

| Command | Description | Utility | More Information |
|--------|----------------|--------|----------------|
| `kubectl get pods` | Lists all existing pods in current namespace. | Use this command to list all pods which are currently running. You can list all pods in a specific namespace with `-n <name_space>`. This can be used to identify any pods that are non-operational. | |
| `kubectl describe pods` | Lists detailed information about pods in your | | |
| `kubectl debug` | Creates a copy of a pod that does terminate when an error is encountered. | | 
| `kubectl logs <pod_name>` | 

## Infrastructure Layer Debugging
The Infrastructure Layer covers nodes and networks in Kubernetes.

| Command | Description | Utility | More Information |
|--------|----------------|--------|----------------|
| `kubectl get nodes` | 
| `kubectl describe node <node_name>` | 
| `kubectl get endpoints <service_name>` | 
| `kubectl get services` 
| `kubectl exec <pod_name> -- nslookup <service_name>` | ??

## Control Plane Layer Debugging
The Control Plane Layer covers cluster management

| Command | Description | Utility | More Information |
|--------|----------------|--------|----------------|
| `kubectl cluster-info dump` | List detailed information about the overall health of your cluster | 
| `kubectl get events` | 

-->
