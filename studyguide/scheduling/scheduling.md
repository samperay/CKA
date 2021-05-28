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
kubectl get pods -l app=nginx
kubectl get pods --selector app=nginx,type=front-end
```

## Annotations
While labels and selectors are used to group objects, annotations are used to record other details for informative purpose

# Taints and Tolerations

Pod to node relationship and how you can restrict what pods are placed on what nodes.
Taints and Tolerations are used to set restrictions on what pods can be scheduled on a node.

pods which are tolerant to particular taint will be scheduled on that node.
there are 3 policies of taints:

The taint effect defines what would happen to the pods if they do not tolerate the taint.

- NoSchedule
- PreferNoSchedule
- NoExecute

The default policy of the any node is no taints so that any pod can be placed in any of the nodes.

Default, master nodes have a **NoSchedule** so that there won't be any pods running on the master nodes.
We would for the testing purpose *untaint* the node and then *revert* back

```
kubectl taint nodes master node-role.kubernetes.io/master- [ No taints and new pods will be created on this node ]
kubectl taint nodes master node-role.kubernetes.io/master:NoSchedule [ tainted ]
```
Few more,
```
kubectl taint nodes kubenode01 env=qa:NoSchedule
kubectl describe nodes kubenode01 | grep -i taint
kubectl taint nodes kubenode01 env-
kubectl describe nodes kubenode01 | grep -i taint
kubectl taint nodes master node-role.kubernetes.io/master:NoSchedule-
```

Taints and Tolerations does not tell the pod to go to a particular node. Instead, it tells the node to only accept pods with certain tolerations.

# NodeSelector

We add new property called Node Selector to the spec section and specify the label. The scheduler uses these labels to match and identify the right node to place the pods onto the nodes.

Label the nodes to ensure your pod is getting scheduled on a desired right nodes by kube-scheduler, however you can't really guarantee that the pods does only gets scheduled on the label being added. sometimes it might also get scheduled to other nodes as well. So, to fix this we would add nodeAffinity on the pods sections.

If there are more complex operations(==, !=, etc) are involved then we might not be able to achieve using nodeSelector, so we need to use *nodeAffinity* and *anti-affinity*

default labels of the node
```
get nodes node01 --show-labels
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

If your application within the pod requires more than the default resources, you need to set them in the pod definition file.

```
resources:
     requests:
      memory: "1Gi"
      cpu: "1"
```

## Limits
By default, k8s sets resource limits to 1 CPU and 512Mi of memory
You can set the resource limits in the pod definition file.
```
limits:
  memory: "2Gi"
  cpu: "2"
```
Note: Remember Requests and Limits for resources are set per container in the pod.

# DaemonSet

DaemonSets are like replicaSet, as it helps in to deploy multiple instances of pod. But it runs one copy of your pod on each node in your cluster.

Daemonset use cases:
- Monitoring solution
- log viewer
- Helper pods for applications
- Networking pods (weave net)

# Static pods
When you need to create pods without any interference from the kubeapi server or kubernetes management control plane, you can write your yaml files and place in the kubelet directory and create pods. these kind of pods which have no interference from the any of the kubernetes components are called as *static pod*

What if we run the static pods and if these nodes are part of kubernetes manage control plane, would it be known to kubeapi server? Yes kubeapi server would be known that there is pod running in the node as kubelet provides a mirror object in the kubeapi as read-only object.


## First Method

You can configure the kubelet to read the pod definition files from a directory on the server designated to store information about pods.The designated directory can be any directory on the host and the location of that directory is passed in to the kubelet as an option while running the service. *--pod-manifest-path*

```
systemctl status kubelet.service
cat /lib/systemd/system/kubelet.service
ExecStart=/usr/local/bin/kubelet \\
.
.
--pod-manifest-path=/etc/kubernetes/manifests
.
```

Create a static pod
```
kubectl run --restart=Never --image=busybox static-busybox --dry-run=client -o yaml --command -- sleep 1000 > /etc/kubernetes/manifests/static-busybox.yaml
```

## Second Method
Incase if you have created kubeadmin way of configuring the cluster, your manifests will be in below location.
```
systemctl status kubelet.service
cat /var/lib/kubelet/config.yaml

look for "staticPodPath: /etc/kubernetes/manifests"
```
