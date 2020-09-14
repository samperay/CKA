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
