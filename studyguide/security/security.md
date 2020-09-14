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
