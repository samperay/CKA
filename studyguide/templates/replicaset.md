---
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name:
  labels:
    app:
    type:
spec:
  template:
    # start of pod definition file
    metadata:
      name:
      labels:
        app:
        type:
    spec:
      containers:
        - name:
          image:
    # end of pod definition file
  replicas: 2
  selector:
    matchLabels:
      type:
