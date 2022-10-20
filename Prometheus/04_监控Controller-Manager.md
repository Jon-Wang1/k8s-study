#### 保证在controller-manager的启动配置文件中有以下两行
```text
      --authentication-kubeconfig=/etc/kubernetes/controller-manager.kubeconfig \
      --authorization-kubeconfig=/etc/kubernetes/controller-manager.kubeconfig \
```
#### 监控Controller-manager
查看ServiceMonitor，添加对应的svc和endpoint。
```yaml
cat > controller-manager-svc.yaml <<EOF
---
apiVersion: v1
kind: Endpoints
metadata:
  labels:
    app.kubernetes.io/name: kube-controller-manager
  name: kube-controller-manager-prom
  namespace: kube-system
subsets:
- addresses:
  - ip: 10.1.1.101
  - ip: 10.1.1.102
  - ip: 10.1.1.103
  ports:
  - name: https-metrics
    port: 10257
    protocol: TCP
---
apiVersion: v1
kind: Service
metadata:
  labels:
    app.kubernetes.io/name: kube-controller-manager
  name: kube-controller-manager-prom
  namespace: kube-system
spec:
  ports:
  - name: https-metrics
    port: 10257
    protocol: TCP
    targetPort: 10257
  sessionAffinity: None
  type: ClusterIP
EOF
kubectl apply -f controller-manager-svc.yaml
```
更改ServiceMonitor配置端口和scheme和Service一致
```shell
kubectl edit servicemonitor kube-controller-manager -n monitoring
```
```text
    - action: drop
      regex: etcd_(debugging|disk|request|server).*
      sourceLabels:
      - __name__
    port: https-metrics
    scheme: https
    tlsConfig:
```
