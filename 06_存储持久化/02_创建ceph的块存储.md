## 创建块存储
块存储一般用于一个Pod挂载一块存储使用，相当于一个服务器新挂了一个盘，只给一个应用使用。两个pod是挂载不了的！  

```shell
cd /root/rook/deploy/examples/csi/rbd
kubectl apply -f storageclass.yaml
```
此yaml文件中包含了两个对象，第一个先创建ceph存储池，第二个创建storageCalss。  
```yaml
apiVersion: ceph.rook.io/v1  
kind: CephBlockPool  
metadata:  
  name: replicapool  # 此名称就是在ceph集群中创建的pool的名称  
  namespace: rook-ceph # namespace:cluster  
spec:  
  failureDomain: host # 以host为单位，将数据的副本分别存在不同的host上，提高安全性。可选osd。 
  replicated:  
    size: 3  # 副本个数，不能大于OSD的个数  
---
apiVersion: storage.k8s.io/v1  
kind: StorageClass  
metadata:  
  name: rook-ceph-block # 创建SC的名称，使用的是块存储    
...  
  #dataPool: ec-data-pool  
  pool: replicapool  # 选择ceph集群中创建的pool的名称    
...  
#mounter: rbd-nbd  
allowVolumeExpansion: true # 开启允许扩容功能
reclaimPolicy: Delete # 回收策略
```

### 在statefulset中使用volumeClaimTemplates，会为每个pod创建一个pvc
```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: web
spec:
  selector:
    matchLabels:
      app: nginx # 必须匹配 .spec.template.metadata.labels
  serviceName: "nginx"
  replicas: 3 # 默认值是 1
  minReadySeconds: 10 # 默认值是 0
  template:
    metadata:
      labels:
        app: nginx # 必须匹配 .spec.selector.matchLabels
    spec:
      terminationGracePeriodSeconds: 10
      containers:
      - name: nginx
        image: nginx
        ports:
        - containerPort: 80
          name: web
        volumeMounts:
        - name: www
          mountPath: /usr/share/nginx/html
  volumeClaimTemplates:
  - metadata:
      name: www
    spec:
      accessModes: [ "ReadWriteOnce" ]
      storageClassName: "rook-ceph-block"
      resources:
        requests:
          storage: 1Gi
```
