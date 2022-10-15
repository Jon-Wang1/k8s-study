### 安装rook
官网链接 https://rook.github.io/docs/rook/  
使用rook-ceph的流程：  
1. 使用crds.yaml，common.yaml，operator.yaml这三个文件搭建rook环境
2. 在K8S的rook环境下创建使用cluster.yaml创建ceph集群
3. 使用storageclass.yaml中的CephBlockPool对象在ceph集群中创建块存储池
4. 再使用storageclass.yaml中的StorageClass创建SC，SC引用了上一步创建的块存储池
5. 再创建PVC，调用上一步创建的SC 

### 下载相关文件
```shell
git clone --single-branch --branch v1.10.3  https://github.com/rook/rook.git
cd rook/deploy/examples
```

### operator.yaml文件需要修改
1. 将operator.yaml中的ROOK_ENABLE_DISCOVERY_DAEMON字段设置为true，即可以在添加新磁盘时可以发现加入到集群。
2. 将以镜像地址修改为本地地址(可选)  
  ROOK_CSI_CEPH_IMAGE: "quay.io/cephcsi/cephcsi:v3.7.1"  
  ROOK_CSI_REGISTRAR_IMAGE: "registry.k8s.io/sig-storage/csi-node-driver-registrar:v2.5.1"  
  ROOK_CSI_RESIZER_IMAGE: "registry.k8s.io/sig-storage/csi-resizer:v1.4.0"  
  ROOK_CSI_PROVISIONER_IMAGE: "registry.k8s.io/sig-storage/csi-provisioner:v3.1.0"  
  ROOK_CSI_SNAPSHOTTER_IMAGE: "registry.k8s.io/sig-storage/csi-snapshotter:v6.0.1"  
  ROOK_CSI_ATTACHER_IMAGE: "registry.k8s.io/sig-storage/csi-attacher:v3.4.0"

### 安装opertor将ROOK_ENABLE_DISCOVERY_DAEMON字段设置为true
```shell
git clone --single-branch --branch v1.10.3  https://github.com/rook/rook.git
cd rook/deploy/examples
# 将operator.yaml中的ROOK_ENABLE_DISCOVERY_DAEMON字段设置为true，即可以在添加新磁盘时可以发现加入到集群。
kubectl create ns rook-ceph
kubectl apply -f crds.yaml -f common.yaml -f operator.yaml
```


### 创建ceph集群cluster.yaml文件需要修改

### cluster.yaml重点字段  
... 
  dataDirHostPath: /var/lib/rook # 删除集群时可能要清楚此目录 
...  
  skipUpgradeChecks: **true**  # 跳过升级  
...  
  mon:  
    # Set the number of mons to be started. Generally recommended to be 3.  
    # For highest availability, an odd number of mons should be specified.  
    **count: 3**  **# monitor的节点数，生产环境中一定要3个节点以上**  
  mgr:  
    # When higher availability of the mgr is needed, increase the count to 2.  
    # In that case, one mgr will be active and one in standby. When Ceph updates which  
    # mgr is active, Rook will update the mgr services to match the active mgr.  
    **count: 2   # 高可用，生成环境设置成2**  
...
  **ssl: false**  # 如果为true，使用域名访问时，由证书问题  
...
  monitoring:  
    # requires Prometheus to be pre-installed  
    enabled: false  # Prometheus监控是否打开  
... 
  storage: # cluster level storage configuration and selection
    **useAllNodes: false**  # 是否将所有节点当成存储节点，一定要改成false  
    **useAllDevices: false**  # 是否将所有磁盘当成存储磁盘，一定要改成false  
...  
    nodes:  
    - name: "node01.ty.com"  
      devices:  
      - name: "sdb"  
      - name: "sdc"  
      - name: "sdd"  
    - name: "node02.ty.com"  
      devices:  
      - name: "sdb"  
      - name: "sdc"  
      - name: "sdd"  
    - name: "node03.ty.com"  
      devices:  
      - name: "sdb"  
      - name: "sdc"  
      - name: "sdd"  
... 
```shell
kubectl apply -f cluster.yaml
kubectl get cephcluster -n rook-ceph
#使用 kubectl get cephcluster -n rook-ceph检查集群状态
#[root@master01 ~]# kubectl get cephcluster -n rook-ceph
#NAME        DATADIRHOSTPATH   MONCOUNT   AGE   PHASE   MESSAGE                        HEALTH      EXTERNAL
#rook-ceph   /var/lib/rook     3          42m   Ready   Cluster created successfully   HEALTH_OK   
#[root@master01 ~]#

```

### 安装 ceph snapshot 控制器
k8s 1.19 版本以上需要单独安装 snapshot 控制器，才能完成 pvc 的快照功能，所以在此提  
前安装下，如果是 1.19 以下版本，不需要单独安装，直接参考视频即可。  
snapshot  
控制器的部署在集群安装时的 k8s ha install 项目中 ，需要切换到对应分支  
具体文档：https://rook.io/docs/rook/v1.6/ceph-csi-snapshot.html  
```shell
cd /root/k8s-ha-install/
kubectl apply -f snapshotter/ -n kube-system
```

### 安装 ceph 客户端工具
```shell
cd /root/rook/deploy/examples
kubectl create -f toolbox.yaml -n rook-ceph

kubectl get po -n rook-ceph

```
### 使用客户端查看集群状态
```shell
kubectl -n rook-ceph exec -it rook-ceph-tools-7564bb9799-h4j2d  -- bash
ceph status
```
### 查看csidriver
```text
[root@master01 ~]# kubectl get csidriver
NAME                            ATTACHREQUIRED   PODINFOONMOUNT   STORAGECAPACITY   TOKENREQUESTS   REQUIRESREPUBLISH   MODES        AGE
rook-ceph.cephfs.csi.ceph.com   true             false            false             <unset>         false               Persistent   3h27m
rook-ceph.rbd.csi.ceph.com      true             false            false             <unset>         false               Persistent   3h27m
[root@master01 ~]# 
```
### 登录dashboard
### 查看ceph的svc
kubectl get svc -n rook-ceph  
根据本地ingress控制器，创建对应的ingress，或新创建一个svc，绑定node的端口号，使用node的地址访问  
apiVersion: v1
kind: Service
metadata:
  labels:
    app: rook-ceph-mgr
    ceph_daemon_id: a
    rook_cluster: rook-ceph
  name: rook-ceph-mgr-dashboard-np
  namespace: rook-ceph
spec:
  ports:
  - name: http-dashboard
    port: 7000
    protocol: TCP
    targetPort: 8443
  selector:
    app: rook-ceph-mgr
    ceph_daemon_id: a
    rook_cluster: rook-ceph
  sessionAffinity: None
  type: NodePort
### 查看ceph dashboard的默认登录用户名为admin，密码
```shell
kubectl -n rook-ceph get secret rook-ceph-dashboard-password -o jsonpath="{['data']['password']}" | base64 --decode && echo
kubectl -n rook-ceph get secret rook-ceph-dashboard-password -o jsonpath="{['data']['password']}" | base64 --decode && echo
```


