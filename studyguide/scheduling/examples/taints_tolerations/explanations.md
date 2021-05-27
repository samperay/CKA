# Explanations on taints and tolerations

taints are applied on the nodes
tolerations applied on the pods

## taint node for color blue

```
- tolerations:
    key: color
    value: blue
    effect: Equal
    action: NoSchedule
```
```
vagrant@master:~/demos$ kubectl taint node node01 color=blue:NoSchedule
node/node01 tainted
vagrant@master:~/demos$ kubectl create -f blue.yml
replicaset.apps/nginx-blue-v4 created
```
The pods are in pending state because, these are not tolerant to the taints being applied on
the node01 for color blue
```
vagrant@master:~/demos$ kubectl get pods
NAME                  READY   STATUS    RESTARTS   AGE
nginx-blue-v4-9d7ht   0/1     Pending   0          4m30s
nginx-blue-v4-r4zm2   0/1     Pending   0          4m30s
nginx-blue-v4-sgrwr   0/1     Pending   0          4m30s
nginx-blue-v4-v6zlq   0/1     Pending   0          4m30s
vagrant@master:~/demos$
```

## applying tolerations on the red pods for the node01
Now, apply tolerations on the red pod and you would see that the nodes would be able to schedule the pods as it has no taints for the color red.

```
vagrant@master:~/demos$ kubectl create -f red-ver5.yml
replicaset.apps/nginx-red-v5 created
vagrant@master:~/demos$
```

Pods being scheduled on the node01

```
vagrant@master:~/demos$ kubectl get pods -l color=red -o wide
NAME                 READY   STATUS    RESTARTS   AGE   IP             NODE     NOMINATED NODE   READINESS GATES
nginx-red-v5-67wwv   1/1     Running   0          25s   192.168.1.43   node01   <none>           <none>
nginx-red-v5-qbz2f   1/1     Running   0          25s   192.168.1.44   node01   <none>           <none>
vagrant@master:~/demos$
```

Try to remove the taint from the node01

```
vagrant@master:~/demos$ kubectl taint node node01 color-
node/node01 untainted
vagrant@master:~/demos$ kubectl describe node node01 | grep -i taint
Taints:             <none>
vagrant@master:~/demos$
```
