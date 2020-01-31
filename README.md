# CKA
Contains files related to k8 for Certified Kubernetes Administrator certifications and practice

# Notes

You can create your own definition files using the generator using `kubectl run` commands.
https://kubernetes.io/docs/reference/kubectl/conventions/

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

# Create YAML

Don't write yaml as it can be generated using below commands.

## Pod
```
kubectl run --generator=run-pod/v1 nginx --image=nginx -o yaml --dry-run > nginx.yaml
```
## Deployment

```
kubectl create deploy nginx --image=nginx --dry-run -o yaml > nginx-deployment.yaml
```
## ReplicationSet
```
kubectl get rs deployment/nginx -o yaml >nginx-rs.yaml
```

## Service
```
kubectl expose pod hello-world --type=NodePort --name=example-service
kubectl expose deployment hello-world --type=NodePort --name=example-service
```

## Generate From Existing Resources
```
kubectl get deployment "<deployment_name>" -n "<namespace>" -o yaml > new-deployment.yaml
```

# Kubernetes Cheat Sheet
https://kubernetes.io/docs/reference/kubectl/cheatsheet/
