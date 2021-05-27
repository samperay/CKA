---
apiVersion: apps/v1
kind: Deployment
metadata:
  name:
  labels:
    app:
spec:
  replicas: 1
  selector:
    matchLabels:
      app:
  template:
    metadata:
      labels:
        app:
    spec:
      containers:
      - name:
        image:
        ports:
        - containerPort: 80
