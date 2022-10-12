
### 【必看】一些必须的配置更改
将Kube-proxy改为ipvs模式，因为在初始化集群的时候注释了ipvs配置，所以需要自行修改一下：
在master01节点执行
```shell
kubectl edit cm kube-proxy -n kube-system

```
mode: ipvs

### 更新Kube-Proxy的Pod：
```shell
kubectl patch daemonset kube-proxy -p "{\"spec\":{\"template\":{\"metadata\":{\"annotations\":{\"date\":\"`date +'%s'`\"}}}}}" -n kube-system

```
### 验证Kube-Proxy模式
[root@k8s-master01 1.1.1]# curl 127.0.0.1:10249/proxyMode

ipvs

