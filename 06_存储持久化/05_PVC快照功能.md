## PVC 快照
PVC快照功能为K8S的通用功能，只要后端存储支持即可，并不是针对rook-ceph的功能。
### 注意：
PVC快照功能需要 k8s 1.17+
### 块存储快照
#### 创建 snapshotClass
snapshotClass为创建snapshot。
cd /root/rook/deploy/examples/csi/rbd
kubectl apply -f snapshotclass.yaml

#### 创建PVC快照
查看PVC名称
```text
kubectl get pvc
```
#### 将对应的PVC名称写入snapshot.yaml，然后创建快照  
kubectl apply -f snapshot.yaml  
#### 查看快照  
kubectl get volumesnapshot
#### 恢复快照
如果想要创建一个具有某个数据的PVC ，可以从某个快照恢复  
使用下面的yaml文件恢复快照  
注意：dataSource为快照名称，storageClassName为新建pvc的storageClass，storage的大小不能低于原pvc的大小。  
```yaml
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: rbd-pvc-restore
spec:
  storageClassName: rook-ceph-block
  dataSource:
    name: rbd-pvc-snapshot
    kind: VolumeSnapshot
    apiGroup: snapshot.storage.k8s.io
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
```

### 文件系统快照
步骤一样