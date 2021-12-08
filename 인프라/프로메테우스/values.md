```
global:
 storageClass: local-storage
prometheus:
 nodeSelector:
  type: master
 persistence:
  enabled: false
 thanos:
  create: true
  service:
   type: LoadBalancer
   nodePort: 30002
 service:
  type: NodePort
  nodePort: 30001
 scrapeInterval: 30s
 externalLabels:
  cluster: "GS-Cluster"

```



```
objstoreConfig: |-
        #type: s3
        #config:
        #bucket: thanos
        #endpoint: 192.168.150.115:31523
        #access_key: fJyQ816ERv
        #secret_key: jnvmHb9hc6A9qa7Da9FEHpA2WqDIMBoehC66xiXF
        #insecure: true

querier:
 stores:
   - 101.79.1.134:31687
   - 101.79.1.144:30002
   - 101.79.1.152:30002
   - 101.79.1.165:30002
queryFrontend:
 service:
  type: NodePort

storegateway:
  enabled: false

global:
  storageClass: local-path
```



```
helm repo add bitnami https://charts.bitnami.com/bitnami
```

```
helm install -f values.yaml prometheus bitnami/kube-prometheus -n monitoring
helm install -f thanos.yaml thanos bitnami/thanos -n monitoring

helm delete prometheus -n monitoring
```



```
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: local-storage
provisioner: kubernetes.io/no-provisioner
volumeBindingMode: WaitForFirstConsumer
reclaimPolicy: Delete
```



```
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-local
  labels:
    type: local
spec:
  storageClassName: local-storage
  capacity:
    storage: 20Gi
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: "/data"
```



```
kubectl taint nodes --all node-role.kubernetes.io/master-
```

```
kubectl get nodes --show-labels
```

```
kubectl label nodes gedgemgmt01 type=master
```



```
helm repo add influxdata https://helm.influxdata.com/
```

```
helm install influxdb -f invalues.yaml influxdata/influxdb2 -n monitoring
```

```
helm delete influxdb -n monitoring
```

```
 kubectl get pod -n monitoring
 kubectl get pvc -n monitoring
```





```
nodeSelector:
persistence:
  enabled: true
  ## If true will use an existing PVC instead of creating one
  # useExisting: false
  ## Name of existing PVC to be used in the influx deployment
  # name:
  ## influxdb data Persistent Volume Storage Class
  ## If defined, storageClassName: <storageClass>
  ## If set to "-", storageClassName: "", which disables dynamic provisioning
  ## If undefined (the default) or set to null, no storageClassName spec is
  ##   set, choosing the default provisioner.  (gp2 on AWS, standard on
  ##   GKE, AWS & OpenStack)
  ##
  # storageClass: "-"
  accessMode: ReadWriteOnce
  size: 50Gi
  mountPath: /var/lib/influxdbv2
service:
  type: NodePort
  port: 80
  targetPort: 8086
```



```
helm install telegraf-ds --values telegraf.yaml influxdata/telegraf-ds -n monitoring

helm upgrade telegraf-ds --values telegraf.yaml influxdata/telegraf-ds -n monitoring
```

```
override_config:
   toml: |+
     [global_tags]
       foo = "bar"
     [agent]
       interval = "10s"
     [[inputs.mem]]
     [[outputs.influxdb_v2]]
       urls           = ["http://192.168.150.115:32621"]
       bucket         = "default"
       organization   = "influxdata"
       token          = "hUEJW4ACdMD3U3SmvdXZUNbKMEZ53Y04"
```



```
docker-compose down
docker build -t gm-center:0.0.1 .
docker-compose up -d
```



## 순서

텔레그래프

```
kubectl taint nodes --all node-role.kubernetes.io/master-
helm repo add influxdata https://helm.influxdata.com/
helm install telegraf-ds --values telegraf.yaml influxdata/telegraf-ds -n monitoring
```



프로메테우스

```
helm repo add bitnami https://charts.bitnami.com/bitnami
helm install -f values.yaml prometheus bitnami/kube-prometheus -n monitoring
```



```
kubectl apply -f prometheus-pv.yaml
```



스토리지클래스

```
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: local-storage
provisioner: kubernetes.io/no-provisioner
volumeBindingMode: WaitForFirstConsumer
reclaimPolicy: Delete
```

f