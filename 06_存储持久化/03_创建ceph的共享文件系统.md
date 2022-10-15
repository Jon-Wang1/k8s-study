共享文件系统一般用于多个Pod 共享一个存储  
官方文档：https://rook.io/docs/rook/v1.6/ceph filesystem.html  
### 创建共享文件系统存储
```shell
ls -l /root/rook/deploy/examples/filesystem.yaml
cd /root/rook/deploy/examples/
kubectl apply -f filesystem.yaml
```
### 应用后会创建mds的pod
```text
[root@master01 examples]# kubectl get pod -n rook-ceph | grep mds
rook-ceph-mds-myfs-a-5db8c4f8f-rbkst                        1/1     Running     0          3m37s
rook-ceph-mds-myfs-b-65b6bbdcd5-kt95c                       1/1     Running     0          3m36s
[root@master01 examples]# 
```
### 创建storageclass
```shell
cd /root/rook/deploy/examples/csi/cephfs
kubectl apply -f storageclass.yaml
```
### storageclass.yaml
```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: rook-cephfs
# Change "rook-ceph" provisioner prefix to match the operator namespace if needed
provisioner: rook-ceph.cephfs.csi.ceph.com # 使用 kubectl get csidriver查看
parameters:
  # clusterID is the namespace where the rook cluster is running
  # If you change this namespace, also change the namespace below where the secret namespaces are defined
  clusterID: rook-ceph # namespace:cluster

  # CephFS filesystem name into which the volume shall be created
  fsName: myfs  #保持和上一步创建filesystem的名字一样

  # Ceph pool into which the volume shall be created
  # Required for provisionVolume: "true"
  pool: myfs-replicated # 将数据存进myfs-replicated池中
```
### 查看sc
```text
[root@master01 ~]# kubectl get sc
NAME              PROVISIONER                     RECLAIMPOLICY   VOLUMEBINDINGMODE   ALLOWVOLUMEEXPANSION   AGE
rook-ceph-block   rook-ceph.rbd.csi.ceph.com      Delete          Immediate           true                   86m
rook-cephfs       rook-ceph.cephfs.csi.ceph.com   Delete          Immediate           true                   13s
[root@master01 ~]# 
```
### 后续调用这个rook-cephfs名字的SC创建PVC
...
