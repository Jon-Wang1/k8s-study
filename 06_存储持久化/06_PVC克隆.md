### 需要注意的是
1. dataSource的name是被克隆的pvc名称，在此是mysql-pv-claim
2. storageClassName为新建pvc的storageClass名称
3. storage不能小于之前pvc的大小。
```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: rbd-pvc-clone
spec:
  storageClassName: rook-ceph-block
  dataSource:
    name: rbd-pvc
    kind: PersistentVolumeClaim
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
```