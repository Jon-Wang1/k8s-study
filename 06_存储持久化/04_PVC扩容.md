## PVC扩容
* 文件共享类型的PVC扩容需要 k8s 1.15+
* 块存储类型的PVC扩容需要 k8s 1.16+
* PVC扩容需要开启 ExpandCSIVolumes ，新版本的 k8s已经默认打开了这个功能，可以查看自己的 k8s版本是否已经默认打开了该功能
* 如果default为true就不需要打开此功能，如果default为false，需要开启该功能
```text
kube-apiserver -h  |grep ExpandCSIVolumes 
```
### 确认SC的扩容能打开allowVolumeExpansion: true
```text
[root@master01 ~]# kubectl get sc rook-cephfs  -o yaml
allowVolumeExpansion: true
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
```

### 扩容PVC，edit要扩容的PVC的storage字段即可
```shell
kubectl edit pvc cephfs-pvc
```