# Namespace

Namespaces are intended for use in environments with many users spread across multiple teams, or projects. Mainly used for isolation of the resources. 

## Create namespaces CLI

```
kubectl get ns
kubectl create namespace dev
```

Create pod using yaml file, that creates both namespace as well as pod

``` 
kubectl create -f pod.yml
```

```
kubectl run nginx --image=nginx --namespace=dev
kubectl get pods --namespace=dev
```

## Namespace and DNS

When you create a Service, it creates a corresponding DNS entry. This entry is of the form <service-name>.<namespace-name>.svc.cluster.local, which means that if a container just uses <service-name>, it will resolve to the service which is local to a namespace. This is useful for using the same configuration across multiple namespaces such as Development, Staging and Production. If you want to reach across namespaces, you need to use the fully qualified domain name (FQDN).

## Set context 

identify which namespace you need your default namespace to be, then you can set your context accordingly. 

```
kubectl config set-context $(kubectl config current-context) --namespace=default
kubectl config set-context $(kubectl config current-context) --namespace=dev
```