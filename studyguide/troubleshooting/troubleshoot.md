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
