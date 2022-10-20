安装CoreDNS
### 安装官方推荐版本（推荐）
官方镜像地址：coredns/coredns:1.8.6
```shell
cd /root/k8s-ha-install/CoreDNS
# 如果更改了k8s service的网段需要将coredns的serviceIP改成k8s service网段的第十个IP
COREDNS_SERVICE_IP=`kubectl get svc | grep kubernetes | awk '{print $3}'`0
sed -i "s#KUBEDNS_SERVICE_IP#${COREDNS_SERVICE_IP}#g" coredns.yaml

kubectl  create -f coredns.yaml 
```


### 安装最新版CoreDNS ()
```shell
COREDNS_SERVICE_IP=`kubectl get svc | grep kubernetes | awk '{print $3}'`0
cd
git clone https://github.com/coredns/deployment.git
cd deployment/kubernetes
./deploy.sh -s -i ${COREDNS_SERVICE_IP} | kubectl apply -f -

```

