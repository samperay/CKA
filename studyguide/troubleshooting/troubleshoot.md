# Troubleshooting

## Application Failures

- Application/Service status of the webserver
- endpoint of the service and compare it with the selectors
- status and logs of the pod
- logs of the previous pod

```
curl http://web-service-ip:node-port
kubectl describe service web-service
kubectl get pod
kubectl describe pod web
kubectl logs web
kubectl logs web -f --previous
```

## Control Plane Failure

- Nodes in the cluster, are they Ready or NotReady, If they are NotReady then check the LastHeartbeatTime of the node to find out the time when node might have crashed
- Possible CPU and MEMORY using top and df -h
- Status and the logs of the kubelet for the possible issues.
- Check the kubelet Certificates, they are not expired, and in the right group and issued by the right CA

```
kubectl get nodes
kubectl describe node worker-1
top
df -h
serivce kubelet status
sudo journalctl –u kubelet
openssl x509 -in /var/lib/kubelet/worker-1.crt -text
```
## Worker nodes failure

- Status of the Nodes in the cluster, are they Ready or NotReady
- If they are NotReady then check the LastHeartbeatTime of the node
- Check the possible CPU and MEMORY using top and df -h
- Check the status and the logs of the kubelet for the possible issues.
- Check the kubelet Certificates, they are not expired, and in the right group and issued by the right CA.

```
kubectl get nodes
kubectl describe node worker-1
serivce kubelet status
sudo journalctl –u kubelet
openssl x509 -in /var/lib/kubelet/worker-1.crt -text
```
