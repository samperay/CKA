# Services

Kubernetes Services enables communication between various components within an outside of the application.

## Service Types

There are 3 types of service types in kubernetes

- NodePort: Where the services makes an internal POD accessable on a POD on the NODE
- ClusterIP: The service creates a Virtual IP inside the cluster to enable communication between different services such as a set of frontend servers to a set of backend servers
- LoadBalancer: Provisions a loadbalancer for our application in supported cloud providers.

```
kubectl create -f pod-svc.yml
kubectl create -f deployment_svc.yml
kubectl get svc
kubectl describe svc name
kubectl delete -f pod-svc.yml
```