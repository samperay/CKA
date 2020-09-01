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
