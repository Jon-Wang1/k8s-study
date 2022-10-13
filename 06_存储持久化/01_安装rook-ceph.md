### 安装rook
官网链接 https://rook.github.io/docs/rook/
```shell
git clone --single-branch --branch v1.10.3  https://github.com/rook/rook.git
cd rook/deploy/examples
# 将ROOK_ENABLE_DISCOVERY_DAEMON
kubectl create ns rook-ceph
kubectl apply -f crds.yaml -f common.yaml -f operator.yaml
```

### 创建ceph集群
重点更改字段
  storage: # cluster level storage configuration and selection
    **useAllNodes: false
    useAllDevices: false**
    #deviceFilter:
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

```shell

```