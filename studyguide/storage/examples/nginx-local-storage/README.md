# Configure nginx Pod to use PV for storage

# Summary

since, I have only one worker node **node01** you need to create the folder for the nginx root html directory.

```
sudo mkdir /mnt/data
sudo sh -c "echo 'Hello from Kubernetes storage' > /mnt/data/index.html"
cat /mnt/data/index.html
```

- Create PV(pv.yml)
- Create PVC(pvc.yml)
- Create Pod(pod.yml)

Login into container, if **curl** not installed then,

```
apt update
apt install curl
curl http://localhost/
```

else,

```
curl http://localhost/
```


Finally, cleanup

```
kubectl delete -f pod.yml
kubectl delete -f pvc.yml
kubectl delete -f pv.yml

sudo rm -rf /mnt/data/index.html
sudo rmdir /mnt/data
```
