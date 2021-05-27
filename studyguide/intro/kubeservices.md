# Deployment

Modifying, scaling, upgrading the resource allocation all these can be done using deployments.
single instace of application or each container is encapsulated in pod, and such pods are deployed using the replication controller or replica sets. kubernetes provides the higher object called **deployment** which provides us capabilty upgrade instances using rolling update, scaling, modify etc

## Deployment using files

```
kubectl create -f deps.yml
kubectl get deployments
kubectl describe deployment depname
kubectl delete deployment depname
```

## Deployment using kubectl cli
```
kubectl create deployment nginx --image=nginx
kubectl run nginx image=nginx:latest
kubectl create deployment nginx --image=nginx --dry-run=client -o yaml
kubectl create deployment nginx --image=nginx --dry-run=client -o yaml > nginx.yml
```

## list all kubernetes resources

```
kubectl get all
```

# Namespace

Namespaces are intended for use in environments with many users spread across multiple teams, or projects. Mainly used for isolation of the resources.

## Create namespaces CLI

```
kubectl get ns
kubectl create ns dev
```

Create pod using yaml file, that creates both namespace as well as pod

```
kubectl create -f pod.yml
```

```
kubectl run nginx --image=nginx -n dev
kubectl get pods -n dev
```

## Namespace and DNS

When you create a Service, it creates a corresponding DNS entry. This entry is of the form <service-name>.<namespace-name>.svc.cluster.local, which means that if a container just uses <service-name>, it will resolve to the service which is local to a namespace. This is useful for using the same configuration across multiple namespaces such as Development, Staging and Production. If you want to reach across namespaces, you need to use the fully qualified domain name (FQDN).

## Set context

identify which namespace you need your default namespace to be, then you can set your context accordingly.

```
kubectl config set-context $(kubectl config current-context) --namespace=default
kubectl config set-context $(kubectl config current-context) --namespace=dev
```

# Pods

## Deploy pods

```
kubectl run nginx --image=nginx
kubectl get pods -o wide
```

## Reference
- https://kubernetes.io/docs/concepts/workloads/pods/pod/
- https://kubernetes.io/docs/concepts/workloads/pods/pod-overview/
- https://kubernetes.io/docs/tutorials/kubernetes-basics/explore/explore-intro/

# Replication Controller

A ReplicationController ensures that a specified number of pod replicas are running at any one time. In other words, a ReplicationController makes sure that a pod or a homogeneous set of pods is always up and available.

ReplicationController are automatically replaced if they fail, are deleted, or are terminated

Replication is used for the core purpose of Reliability, Load Balancing, and Scaling

```
kubectl create -f replication_controller.yml
kubectl get rc
kubectl describe rc
```

# Replication Set
Replica Set ensures how many replica of pod should be running. It can be considered as a replacement of replication controller.

```
kubectl create -f replicaset.yml
kubectl get rs
kubectl describe rs
```

# Difference between replication controller and replication set
The key difference between the replica set and the replication controller is, the replication controller only supports equality-based selector whereas the replica set supports set-based selector
https://blog.knoldus.com/replicationcontroller-and-replicaset-in-kubernetes/

# Services

Kubernetes Services enables communication between various components within an outside of the application.

## Service Types

There are 3 types of service types in kubernetes

- NodePort: Where the services makes an internal POD accessible on a POD on the NODE
- ClusterIP: The service creates a Virtual IP inside the cluster to enable communication between different services such as a set of frontend servers to a set of backend servers
- LoadBalancer: Provisions a load balancer for our application in supported cloud providers.

```
kubectl create -f pod-svc.yml
kubectl create -f deployment_svc.yml
kubectl get svc
kubectl describe svc name
kubectl delete -f pod-svc.yml
```
