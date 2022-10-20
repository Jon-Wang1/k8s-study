#### 监控scheduler
查看ServiceMonitor，添加对应的svc和endpoint。
```yaml
cat > scheduler-svc.yaml <<EOF
---
apiVersion: v1
kind: Endpoints
metadata:
  labels:
    app.kubernetes.io/name: kube-scheduler
  name: kube-scheduler-prom
  namespace: kube-system
subsets:
- addresses:
  - ip: 10.1.1.101
  - ip: 10.1.1.102
  - ip: 10.1.1.103
  ports:
  - name: https-metrics
    port: 10259
    protocol: TCP
---
apiVersion: v1
kind: Service
metadata:
  labels:
    app.kubernetes.io/name: kube-scheduler
  name: kube-scheduler-prom
  namespace: kube-system
spec:
  ports:
  - name: https-metrics
    port: 10259
    protocol: TCP
    targetPort: 10259
  sessionAffinity: None
  type: ClusterIP
EOF
kubectl apply -f scheduler-svc.yaml
```
更改ServiceMonitor配置端口和scheme和Service一致
```shell
kubectl edit servicemonitor kube-scheduler -n monitoring
```
