Kiali为 Istio 提供了可视化的界面，可以在 Kiali 上进行观测流量的走向、调用链，同时还可
以使用 Kiali 进行配置管理，给用户带来了很好的体验。  
接下来在 Kubernetes 中安装 Kiali 工具，首先进入到 Istio 的安装包目录：
```shell
kubectl create -f samples/addons/kiali.yaml
kubectl get pod,svc -n istio-system -l app=kiali
kubectl create ingress kiali --rule=kiali.ty.com/*=kiali:20001 -n istio-system
```

### 安装jaeger

除了Kiali之外，还需要一个链路追踪的工具，安装该工具可以在Kiali的Workloads页面，查看某个服务的Traces信息。  
直接安装即可： 
```shell
kubectl create -f samples/addons/jaeger.yaml
```

### 部署Prometheus并监控Istio

```shell
kubectl create -f samples/addons/prometheus.yaml -f samples/addons/grafana.yaml

```
