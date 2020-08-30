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

# Difference betweek replication controller and replication set
The key difference between the replica set and the replication controller is, the replication controller only supports equality-based selector whereas the replica set supports set-based selector