# CKA
Contains files related to k8 for Certified Kubernetes Administrator certifications and practice

# Notes

You can create your own definition files using the generator using `kubectl run` commands.
https://kubernetes.io/docs/reference/kubectl/conventions/

```
Create an NGINX Pod

kubectl run --generator=run-pod/v1 nginx --image=nginx

Generate POD Manifest YAML file (-o yaml). Don't create it(--dry-run)
kubectl run --generator=run-pod/v1 nginx --image=nginx --dry-run -o yaml



Create a deployment
kubectl run --generator=deployment/v1beta1 nginx --image=nginx

Generate Deployment YAML file (-o yaml). Don't create it(--dry-run)
kubectl run --generator=deployment/v1beta1 nginx --image=nginx --dry-run -o yaml

Generate Deployment YAML file (-o yaml). Don't create it(--dry-run) with 4 Replicas (--replicas=4)
kubectl run --generator=deployment/v1beta1 nginx --image=nginx --dry-run --replicas=4 -o yaml

Save it to a file - (If you need to modify or add some other details before actually creating it)
kubectl run --generator=deployment/v1beta1 nginx --image=nginx --dry-run --replicas=4 -o yaml > nginx-deployment.yaml

```

