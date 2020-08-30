# kubernetes Cluster Architecture

## Architecture Components
Kubernetes consists of nodes which are physcial or cloud which hosts applications on the nodes in form of containers. 

The master node in the Kubernetes cluster is responsible for managing the Kubernetes cluster storing information regarding the different nodes planning which containers cause where monitoring the notes and containers on them etc. 

The Master node does all of these using a set of components together known as the *control plane components.*

## etcd
details about the container informations like when it was loaded, which container is placed on which nodes etc. These are stored as DB in key/value format.

- The ETCD Datastore stores information regarding the cluster such as Nodes, PODS, Configs, Secrets, Accounts, Roles, Bindings and Others. 
- Every information you see when you run the kubectl get command is from the ETCD Server. 

## kube-api
The kube-apiserver is the primary management component of kubernetes. The kube-api server is responsible for
orchestrating all operations within the cluster. it exposes kubernetes API which is used by external users to perform mgmt operations on the cluster as well as the various controllers to monitor the state of the cluster and make the necessary changes as required and by the worker nodes to communicate with the server.

Kube-apiserver is responsible for authenticating, validating requests, retrieving and Updating data in ETCD key-value store. In fact kube-apiserver is the only component that interacts directly to the etcd datastore. The other components such as kube-scheduler, kube-controller-manager and kubelet uses the API-Server to update in the cluster in thier respective areas.

## Control Managers
Kube Controller Manager manages various controllers in kubernetes.

In kubernetes terms, a controller is a process that continously monitors the state of the components within the system and works towards bringing the whole system to the desired functioning state.

  - *Node-controller* : The node-controller takes care of nodes. They're responsible for onboarding new nodes to
the cluster handling situations where nodes become unavailable or get gets destroyed. 
    - Monitoring Nodes
      - Node Monitor Period 5s
      - Node Monitoring Grace Period Status 40s
      - POD Eviction timeout 5m

  - *Replication-Controller* : Ensures that the desired number of containers are running at all times in your replication group. 


## kube-scheduler
kube-scheduler is responsible for scheduling pods on nodes.The kube-scheduler is only responsible for deciding which pod goes on which node. It doesn't actually place the pod on the nodes, thats the job of the kubelet.
- Decision factors
  - Resource requirements and Limits
  - Taints and Tolerations
  - Node Selectors and Affinity

## kubelet
Kubelet is the sole point of contact for the kubernetes cluster

A kubelet is an agent that runs on each node in a cluster. It listens for instructions from the kube-api server and deploys or destroys containers on the nodes as required. The kube-api server periodically fetches status reports from the kubelet to monitor the state of nodes and containers on them.

## kube proxy
Within Kubernetes Cluster, every pod can reach every other pod, this is accomplish by deploying a pod networking cluster to the cluster.
Kube-Proxy is a process that runs on each node in the kubernetes cluster.

The Kube-proxy service ensures that the necessary rules are in place on the worker nodes to allow the containers running on them to reach each other.

## Communication on kubernetes mgmt plane
when user types *kubectl get nodes*, kube-api server authenticates the requests, validates it and gets information by *etcd* which stores all the dats using key/value pair. when new pod is created, kubeapi server authenticates first, and then validated. it then creates a pod without assigning it to a node, updates information in *etcd* and *etcd* reverts back to kube-api that information has been updated. 

Now, you have a container which is not assigned to any node, this is being seen by *kube-scheduler*  which verifies the requirements of container and would assign to right nodes and it would update the *kube-api* as to which nodes this container has to be scheduled, then *kube-api* would be send this request to *etcd* to update the inforamtion as to which nodes this container needs to be scheduuled, after which *kube-api* would pass this information to *kubelet* to start the container in appropriate worker node. kubelet then creates pod on the node and instructs the container runtime engine to deploy the image. Once done *kubelet* updates back inforamtion to *kube-api* which then sends back information to *etcd*