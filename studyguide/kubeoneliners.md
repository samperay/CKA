# Kubernetes Commands

This document helps us in understanding using the kubernetes in the imperative way.

It would be helpful if we are creating alias for long commands to save time during exams..
here is the list, I shall be using in this repository.

```
cat ~/.bash_aliases

# kubectl get
alias k="kubectl"
alias kgn="kubectl get nodes -o wide"
alias kgp="kubectl get pods -o wide"
alias kgd="kubectl get deployment -o wide"
alias kgs="kubectl get svc -o wide"

# kubectl describe
alias kdp="kubectl describe pod"
alias kdd="kubectl describe deployment"
alias kds="kubectl describe service"
alias kdn="kubectl describe node"
```

## Imperative Method

```
kubectl run nginx --image=nginx:latest
kubectl create deployment --image=nginx nginx
kubectl expose deployment nginx --port 80
kubectl edit deployment nginx
kubectl scale deployment nginx --replicas=5
kubectl set image deployment nginx nginx=nginx:1.15.9
kubectl create -f nginx.yml
kubectl replace -f nginx.yml
kubectl delete -f nginx.yml
kubectl run nginx --image=nginx  --dry-run=client -o yaml
kubectl create deployment --image=nginx nginx --dry-run=client -o yaml > nginx-deployment.yaml
kubectl expose pod redis --port=6379 --name redis-service --dry-run=client -o yaml
                                or
kubectl create service clusterip redis --tcp=6379:6379 --dry-run=client -o yaml
kubectl expose pod nginx --port=80 --name nginx-service --type=NodePort --dry-run=client -o yaml
                                or
kubectl create service nodeport nginx --tcp=80:80 --node-port=30080 --dry-run=client -o yaml
```

## Declarative
```
kubectl apply -f nginx.yml
```

# Kubernetes Cheat Sheet
https://kubernetes.io/docs/reference/kubectl/cheatsheet/
