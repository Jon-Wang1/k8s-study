### 安装Metrics Server
在新版的Kubernetes中系统资源的采集均使用Metrics-server，可以通过Metrics采集节点和Pod的内存、磁盘、CPU和网络的使用率。

```shell
cd /root/k8s-ha-install/metrics-server
kubectl  create -f . 
```

### 等待metrics server启动然后查看状态
kubectl  top node
NAME           CPU(cores)   CPU%   MEMORY(bytes)   MEMORY%   
k8s-master01   231m         5%     1620Mi          42%       
k8s-master02   274m         6%     1203Mi          31%       
k8s-master03   202m         5%     1251Mi          32%       
k8s-node01     69m          1%     667Mi           17%       
k8s-node02     73m          1%     65
