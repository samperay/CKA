# Taints && Tolerations

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
# Node Affinity
**Goal:** only *blue* pods to be scheduled on the *node01*

1) tainted, *master* for **color=blue** so that none of the blue pods scheduled on the master node.. i.e red pods would be assigned only on the master nodes.
```
vagrant@master:~/demos$ kubectl describe node master | grep -i taint
Taints:             color=blue:NoSchedule
vagrant@master:~/demos$
```

```
vagrant@master:~/demos$ kubectl create -f red-ver5.yml
replicaset.apps/nginx-red-v5 created
vagrant@master:~/demos$ kubectl get pods -o wide
NAME                  READY   STATUS    RESTARTS   AGE   IP             NODE     NOMINATED NODE   READINESS GATES
nginx-blue-v4-csnr2   1/1     Running   0          67m   192.168.1.69   node01   <none>           <none>
nginx-blue-v4-gtmbc   1/1     Running   0          67m   192.168.1.70   node01   <none>           <none>
nginx-blue-v4-pw4l7   1/1     Running   0          67m   192.168.1.71   node01   <none>           <none>
nginx-blue-v4-xfvwf   1/1     Running   0          67m   192.168.0.22   master   <none>           <none>
nginx-red-v5-4fb4w    1/1     Running   0          17s   192.168.0.26   master   <none>           <none>
nginx-red-v5-f2zxd    1/1     Running   0          17s   192.168.0.24   master   <none>           <none>
nginx-red-v5-fhrzp    1/1     Running   0          17s   192.168.0.27   master   <none>           <none>
nginx-red-v5-thdg4    1/1     Running   0          17s   192.168.0.23   master   <none>           <none>
nginx-red-v5-z7d47    1/1     Running   0          17s   192.168.0.25   master   <none>           <none>
vagrant@master:~/demos$
```

2) tolerations, added to the **blue** pod so that blue pods won't be scheduled on the master node as taint is applied on the **master**
```
spec:
  containers:
    - name: nginx
      image: sunlnx/kubenginx:v4
  tolerations:
    - key: color
      value: blue
      operator: Equal
      effect: NoSchedule
```

However, those two above won't guarantee that the pods would be only on node01, hence we use nodeAffinity
I tried to add the label for the node *node01* for the color *blue*.
```      
vagrant@master:~/demos$ kubectl get nodes node01 --show-labels
NAME     STATUS   ROLES    AGE    VERSION   LABELS
node01   Ready    <none>   124d   v1.20.2   beta.kubernetes.io/arch=amd64,beta.kubernetes.io/os=linux,color=blue,kubernetes.io/arch=amd64,kubernetes.io/hostname=node01,kubernetes.io/os=linux
vagrant@master:~/demos$
```

Despite *tolerations* being added on the pods, you would still be able to get the pods that could be assigned on the other master nodes.

```
vagrant@master:~/demos$ kubectl get pods -o wide
NAME                  READY   STATUS    RESTARTS   AGE    IP             NODE     NOMINATED NODE   READINESS GATES
nginx-blue-v4-csnr2   1/1     Running   0          69m    192.168.1.69   node01   <none>           <none>
nginx-blue-v4-gtmbc   1/1     Running   0          69m    192.168.1.70   node01   <none>           <none>
nginx-blue-v4-pw4l7   1/1     Running   0          69m    192.168.1.71   node01   <none>           <none>
nginx-blue-v4-xfvwf   1/1     Running   0          69m    192.168.0.22   master   <none>           <none> <<============
nginx-red-v5-4fb4w    1/1     Running   0          3m1s   192.168.0.26   master   <none>           <none>
nginx-red-v5-f2zxd    1/1     Running   0          3m1s   192.168.0.24   master   <none>           <none>
nginx-red-v5-fhrzp    1/1     Running   0          3m1s   192.168.0.27   master   <none>           <none>
nginx-red-v5-thdg4    1/1     Running   0          3m1s   192.168.0.23   master   <none>           <none>
nginx-red-v5-z7d47    1/1     Running   0          3m1s   192.168.0.25   master   <none>           <none>
vagrant@master:~/demos$
```

Adding nodeAffinity to the blue pods would ensure that all the pods would be on the **node01**

```
affinity:
  nodeAffinity:
    requiredDuringSchedulingIgnoredDuringExecution:
      nodeSelectorTerms:
        - matchExpressions:
          - key: color
            operator: In
            values:
            - blue
```

Once, you create pods, you will have your pod running on the node01 only.

```
vagrant@master:~/demos$ kubectl create -f red-ver5.yml
replicaset.apps/nginx-red-v5 created
vagrant@master:~/demos$ kubectl create -f blue-ver4.yml
replicaset.apps/nginx-blue-v4 created
vagrant@master:~/demos$

vagrant@master:~/demos$ kubectl get pods -o wide
NAME                  READY   STATUS    RESTARTS   AGE   IP             NODE     NOMINATED NODE   READINESS GATES
nginx-blue-v4-56kwg   1/1     Running   0          63s   192.168.1.87   node01   <none>           <none>
nginx-blue-v4-8jpdh   1/1     Running   0          63s   192.168.1.88   node01   <none>           <none>
nginx-blue-v4-fpmqd   1/1     Running   0          63s   192.168.1.86   node01   <none>           <none>
nginx-blue-v4-fsvq4   1/1     Running   0          63s   192.168.1.89   node01   <none>           <none>
nginx-red-v5-46nlq    1/1     Running   0          67s   192.168.0.45   master   <none>           <none>
nginx-red-v5-cpltl    1/1     Running   0          67s   192.168.1.83   node01   <none>           <none>
nginx-red-v5-nvrzd    1/1     Running   0          67s   192.168.0.46   master   <none>           <none>
nginx-red-v5-z28t2    1/1     Running   0          67s   192.168.1.85   node01   <none>           <none>
nginx-red-v5-z5nbw    1/1     Running   0          67s   192.168.1.84   node01   <none>           <none>
vagrant@master:~/demos$
```
