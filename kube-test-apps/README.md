# nginx

This deployment file is used to only test if the created kubernetes cluster is working as expected.
So, we would try to execute this below command and if everything is working as expected, then we would be able to get the webpage of the nginx container in the browser.

This below yaml file creates, deployment as well as expose service as a NodePort.

```
kubectl create -f nginx.yml
minikube service nginx-service --url
```
Copy and paste the URL in your browser, it should default to nginx webpage.
