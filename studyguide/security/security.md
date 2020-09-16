# Security

We need to make two types of decisions.

*Authentication* Who can access API server
*Authorization* What can they do by gaining access to the cluster.

## TLS Certificates

All communication with the cluster, between the various components such as the ETCD Cluster, kube-controller-manager, scheduler, api server.

Communication between the applications within cluster is defined by *network policies*

## Authentication

there are two types of users,

- users ( admin/developer)
- service accounts (third party for service integrations, bots )

All the user access is managed by API server and all requests go through it.

### Authorization Mechanism

- static file
- static token file
- certificates
- identity services

- authenticate using basic user file credentials
```
curl -v -k http://master-node-ip:6443/api/v1/pods -u "user1:password123"
```
- authenticate using token
```
curl -v -k http://master-node-ip:6443/api/v1/pods --header "Authorization: Bearer <Token>"
```
static file and token would be the easiest but not recommended, instead use *role based access control*

## TLS Certificates

- What are TLS certificates?
- How does kubernetes use certificates?
- How to generate them?
- How to configure them?
- How to view them?
- How to troubleshoot issues related to certificates

A certificate is used to gurantee trust between 2 parties during a transaction. tls certificates ensure that the communication between them is encrypted.

### Symmetric

It uses the same key to encrypt and decrypt the data and the key has to be exchanged between the sender and the receiver

### Asymmetric

Instead of using single key to encrpyt and decrypt data, asymmetric encryption uses a pair of keys, a private key and a public key.

## Certificate naming conventions
*certificate public key* *.crt, *.pem
*Certificate private key* *.pem

## Kubernetes TLS Certificates
Since, the method is being used for creation cluster is *kubeadm*, all the certificates are stored in below localtion.
*/etc/kubernetes/pki/*

Some of the certitcates are listed below

Server Certificates for Servers
Client Certificates for Clients

Public keys
```
apiserver-etcd-client.crt  
apiserver-kubelet-client.crt  
apiserver.crt
ca.crt
front-proxy-ca.crt  
front-proxy-client.crt
```

Private keys
```
apiserver-etcd-client.key  
apiserver-kubelet-client.key  
apiserver.key
ca.key
front-proxy-ca.key  
front-proxy-client.key  
sa.key
```

Viewing, certiticate details, these below to be looked into certificates

```
openssl x509 -in /etc/kubernetes/pki/apiserver.crt -text -noout
```

- Issuer
- Validity
- Subject
- Subject Alternative Names

Troubleshooting, kubeadm installation method.
```
kubectl logs etcd-master
docker ps -a
docker logs <container-id>
```

## Certificate API

The CA is really just the pair of key and certificate files that we have generated, whoever gains access to these pair of files can sign any certificate for the kubernetes environment.
Kubernetes has a built-in certificates API that can do this for you.

- CertificateSigningRequest
- Review Requests
- Approve Requests
- Share to Users

So user does create a key and CSR
```
openssl genrsa -out jane.key 2048
openssl req -new -key jane.key -subj "/CN=jane" -out jane.csr
```

Sends the request to the administrator and the adminsitrator takes the key and creates a CSR object, with kind as "CertificateSigningRequest" and a encoded "jane.csr"

```
apiVersion: certificates.k8s.io/v1beta1
kind: CertificateSigningRequest
metadata:
  name: jane
spec:
  groups:
  - system:authenticated
  usages:
  - digital signature
  - key encipherment
  - server auth
  request:
    <certificate-goes-here>
```

```
kubectl get csr
kubectl certificate approve jane
kubectl get csr jane -o yaml
echo "<certificate>" |base64 --decode
```

Deny request if its inappropriate
```
kubectl certificate deny jane
kubectl delete csr jane
```

All the certificate releated operations are carried out by the controller manager.
```
$ cat kube-controller-manager.yaml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    component: kube-controller-manager
    tier: control-plane
  name: kube-controller-manager
  namespace: kube-system
spec:
  containers:
  - command:
    - kube-controller-manager
<Clip>
    - --cluster-signing-cert-file=/etc/kubernetes/pki/ca.crt
    - --cluster-signing-key-file=/etc/kubernetes/pki/ca.key

```

## kubeconfig

Client uses the certificate file and key to query the kubernetes Rest API for a list of pods using curl.

```
kubectl get pods --kubeconfig config
```

The kubeconfig file has 3 sections

- Clusters
- Contexts
- Users

```
apiVersion: v1
clusters: <============
- cluster:
    certificate-authority-data: LS0tLS1g==
    server: https://192.168.56.51:6443
  name: kubernetes
contexts:  <============
- context:
    cluster: kubernetes
    user: kubernetes-admin
  name: kubernetes-admin@kubernetes
current-context: kubernetes-admin@kubernetes
kind: Config
preferences: {}
users: <============
- name: kubernetes-admin
  user:
    client-certificate-data: LS0tLtLQo=
    client-key-data: LS0tCg==
```

```
kubectl config view
kubectl config veiw --kubeconfig=my-custom-config
kubeclt config -h
```

## API Groups

The kubernetes API is grouped into multiple such groups based on thier purpose. Such as one for APIs, one for healthz, metrics and logs etc.

These APIs are catagorized into two.
- core group, functionality exists
- Named group, More organized and going forward all the newer features are going to be made available

```
curl http://localhost:6443 -k
curl http://localhost:6443/apis -k
```

## RBAC

### Role

Roles and Rolebindings are namespaced meaning they are created within namespaces.

Roles can be described with below three parameters.
- apiGroups
- resources
- verbs

we need to link the user to that role. i.e called *rolebinding*

```
kubectl create -f developer-role.yaml
kubectl create -f devuser-developer-binding.yaml
kubectl get roles
kubectl get rolebindings
kubectl describe role developer
kubectl describe rolebinding devuser-developer-binding
```

## Verifiy Access
```
kubectl auth can-i create deployments
kubectl auth can-i delete nodes
kubectl auth can-i create deployments --as dev-user
kubectl auth can-i create pods --as dev-user
kubectl auth can-i create pods --as dev-user --namespace test
```

## Cluster Roles

You won't be able to isolate the nodes within the namespaces, as these are cluster wide resources.
there by, resources are catagorized either namespaced or cluster

```
kubectl api-resources --namespaced=true
kubectl api-resources --namespaced=false
```

You can create a cluster role for namespace resources as well. When you do that user will have access to these resources across all namespaces

```
kubectl create -f cluster-roles.yml
kubectl auth can-i list nodes --as michelle
```

## Images Security

when we pull image from "nginx" we are pulling from the nginx/nginx of the private docker.io hub.
so you need to login to your hub.docker.com so that kubelet will try to pull image from the repository and deploy applications in the worker nodes.

```
containers:
- name: nginx
  image: nginx
```

what if the repository is private, then you need to specify the DNS name prefixed with the docker image.
```
containers:
- name: nginx
  image: private-repo/nginx:latest
```

If you have to modify the docker containers which has to be provided with the secrets, kubernets have secrets where you can store your credentials.

```
kubectl create secret docker-registry private-reg-cred --docker-username=dock_user --docker-password=dock_password --docker-server=myprivateregistry.com:5000 --docker-email=dock_user@myprivateregistry.com
```
you would be using these credentials for the pod in applications to be up and running.

```
spec:
  containers:
  - name: private-reg-container
    image: <your-private-image>
  imagePullSecrets:
  - name: private-reg-cred
```

## Secrurity Contexts
You may choose to configure the security settings at a container level or at a pod level.

To add security context on the container and a field called securityContext under the spec section.

```
spec:
  securityContext:
    runAsUser: 1010
  containers:
  - name: ubuntu
```

To add security context at security level,

```
spec:
  containers:
  - name: ubuntu
    image: ubuntu
    command: ["sleep", "3600"]
    securityContext:
      runAsUser: 1000
      capabilities:
        add: ["MAC_ADMIN"]
```

## Network Policies

There are two types of traffic

- Ingress
- Egress

Network policies can be applied on the pods for which traffic can be allowed or rejected.
https://kubernetes.io/docs/concepts/services-networking/network-policies/
