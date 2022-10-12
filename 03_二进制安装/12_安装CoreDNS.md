安装CoreDNS
安装官方推荐版本（推荐）
```shell
cd /root/k8s-ha-install/CoreDNS
# 如果更改了k8s service的网段需要将coredns的serviceIP改成k8s service网段的第十个IP
COREDNS_SERVICE_IP=`kubectl get svc | grep kubernetes | awk '{print $3}'`0
sed -i "s#KUBEDNS_SERVICE_IP#${COREDNS_SERVICE_IP}#g" coredns.yaml

kubectl  create -f coredns.yaml 
```

```shell

```
```shell

```
```shell

```
```shell

```
```shell

```





安装coredns
[root@k8s-master01 k8s-ha-install]# kubectl  create -f CoreDNS/coredns.yaml 
serviceaccount/coredns created
clusterrole.rbac.authorization.k8s.io/system:coredns created
clusterrolebinding.rbac.authorization.k8s.io/system:coredns created
configmap/coredns created
deployment.apps/coredns created
service/kube-dns created
12.2 	 安装最新版CoreDNS
COREDNS_SERVICE_IP=`kubectl get svc | grep kubernetes | awk '{print $3}'`0


git clone https://github.com/coredns/deployment.git
cd deployment/kubernetes
# ./deploy.sh -s -i ${COREDNS_SERVICE_IP} | kubectl apply -f -
serviceaccount/coredns created
clusterrole.rbac.authorization.k8s.io/system:coredns created
clusterrolebinding.rbac.authorization.k8s.io/system:coredns created
configmap/coredns created
deployment.apps/coredns created
service/kube-dns created
查看状态
 # kubectl get po -n kube-system -l k8s-app=kube-dns
NAME                       READY   STATUS    RESTARTS   AGE
coredns-85b4878f78-h29kh   1/1     Running   0          8h
