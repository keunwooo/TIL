# Node Affinity 

```shell
kubectl get nodes --show-labels
```

```shell
kubectl label nodes <your-node-name> disktype=ssd
```

```shell
kubectl get nodes --show-labels
```

```
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
        - matchExpressions:
          - key: disktype
            operator: In
            values:
            - ssd            
  containers:
  - name: nginx
    image: nginx
    imagePullPolicy: IfNotPresent
```



cont=backend,