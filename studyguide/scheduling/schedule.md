# Scheduling

If your kubernetes don't have a scheduler, then every POD has a field called NodeName that by default is not set. Once identified it schedules the POD on the node by setting the *nodeName* property to the name of the node by creating a binding object.

##  NodeName

you can manually set to assign pods to that nodes by setting the property *nodeName* in your pod definition file while you create pod.

```
kubectl create -f nodeName.yml
kubectl delete -f nodeName.yml
```

## Labels & Selectors
Labels and Selectors are standard methods to group things together. Labels are properties attach to each item. Selectors help you to filter these items

you can select the pod using labels
```
kubectl get pods --selector app=nginx
```

## Annotations
While labels and selectors are used to group objects, annotations are used to record other details for informative purpose

# Taints and Tolerations

Pod to node relationship and how you can restrict what pods are placed on what nodes.
Taints and Tolerations are used to set restrictions on what pods can be scheduled on a node.

pods which are tolerant to partitcular taint will be scheduled on that node.
there are 3 policies of tains:

The taint effect defines what would happen to the pods if they do not tolerate the taint.

- NoSchedule
- PreferNoSchedule
- NoExecute1

The default policy of the any node is no tains so that any pod can be placed in any of the nodes.

```
kubectl taint nodes kubenode01 env=qa:NoSchedule
kubectl describe nodes kubenode01 | grep -i taint
kubectl taint nodes kubenode01 env-
kubectl describe nodes kubenode01 | grep -i taint
```

Taints and Tolerations does not tell the pod to go to a particular node. Instead, it tells the node to only accept pods with certain tolerations.

# NodeSelector

We add new property called Node Selector to the spec section and specify the label. The scheduler uses these labels to match and identify the right node to place the pods on

Label the nodes to ensure your pod is getting scheduled on a desired right nodes by kube-scheduler.
If there are more complex operations(==, !=, etc) are involved then we might not be able to achieve using nodeSelector, so we need to use *nodeAffinity* and *antiaffinity*

default labels of the node
```
kubectl describe nodes kubenode01 | grep -i label
Labels:             beta.kubernetes.io/arch=amd64
```

```
kubectl label nodes kubenode01 size=small
```

# Node Affinity
The primary feature of Node Affinity is to ensure that the pods are hosted on particular nodes.

## Node Affinity Types

### Available
requiredDuringSchedulingIgnoredDuringExecution
preferredDuringSchedulingIgnoredDuringExecution

### Planned
requiredDuringSchedulingRequriedDuringExecution
preferredDuringSchedulingRequiredDuringExecution

# Resource & Limits
Each node has a set of CPU, Memory and Disk resources available.
By default, K8s assume that a pod or container within a pod requires 0.5 CPU and 256Mi of memory. This is known as the Resource Request for a container.

If your application within the pod requires more than the default resources, you need to set them in the pod defination file.

```
resources:
     requests:
      memory: "1Gi"
      cpu: "1"
```

## Limits

By default, k8s sets resource limits to 1 CPU and 512Mi of memory

You can set the resource limits in the pod defination file.
```
limits:
  memory: "2Gi"
  cpu: "2"
```
Note: Remember Requests and Limits for resources are set per container in the pod.

# DaemonSet

DaemonSets are like replicasets, as it helps in to deploy multiple instances of pod. But it runs one copy of your pod on each node in your cluster.

Daemonset usecases:
- Monitoring solution
- log viewer
- Helper pods for applications
- Networking pods (weave net)
