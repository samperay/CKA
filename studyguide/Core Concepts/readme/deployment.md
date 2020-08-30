# Deployment

Modifying, scaling, upgrading the resource allocation all these can be done using deployments.
single instace of application or each container is encapsulated in pod, and such pods are deployed using the replication controller or replica sets. kubernetes provides the higher object called **deployment* which provides us capabilty upgrade instances using rolling update, scaling, modify etc

## Deployment using files

```
kubectl create -f deps.yml
kubectl get deployments
kubectl describe deployment depname
kubectl delete deployment depname
```

## Deployment using kubectl cli
```
kubectl create deployment --image=nginx nginx
kubectl run nginx image=nginx:latest
kubectl create deployment --image=nginx --dry-run -o yaml
kubectl create deployment --image=nginx --dry-run -o yaml > nginx.yml
```

## list all kubernetes resources

```
kubectl get all
```