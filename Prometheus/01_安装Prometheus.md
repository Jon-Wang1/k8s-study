## 使用kube-Prometheus stack安装
https://github.com/prometheus-operator/kube-prometheus
```shell
# Create the namespace and CRDs, and then wait for them to be available before creating the remaining resources
kubectl apply --server-side -f manifests/setup
until kubectl get servicemonitors --all-namespaces ; do date; sleep 1; echo ""; done
kubectl apply -f manifests/

kubectl create ingress prometheus --rule=prometheus.ty.com/*=prometheus-k8s:9090 -n monitoring
```