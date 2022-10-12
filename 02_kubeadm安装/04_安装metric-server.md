在新版的Kubernetes中系统资源的采集均使用Metrics-server，可以通过Metrics采集节点和Pod的内存、磁盘、CPU和网络的使用率。

### 将Master01节点的front-proxy-ca.crt复制到所有Node节点
因为metric为一个pod，会随机部署到任意一个node上，并且会使用到这个front-proxy-ca.crt
```shell
scp /etc/kubernetes/pki/front-proxy-ca.crt node01:/etc/kubernetes/pki/front-proxy-ca.crt
scp /etc/kubernetes/pki/front-proxy-ca.crt node02:/etc/kubernetes/pki/front-proxy-ca.crt
scp /etc/kubernetes/pki/front-proxy-ca.crt node03:/etc/kubernetes/pki/front-proxy-ca.crt

```
### 安装metrics server
```shell
cd /root/k8s-ha-install/kubeadm-metrics-server
kubectl  create -f comp.yaml 

```

