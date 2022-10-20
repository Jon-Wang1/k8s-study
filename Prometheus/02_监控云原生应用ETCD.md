## 云原生应用 Etcd 监控
#### 测试访问Etcd Metrics接口：
```shell
curl -s --cert /etc/kubernetes/pki/etcd/etcd.pem \
        --key /etc/kubernetes/pki/etcd/etcd-key.pem \
        https://10.1.1.102:2379/metrics -k | tail -1
```
证书的位置可以在Etcd配置文件中获得（注意配置文件的位置，不同的集群位置可能不同，Kubeadm安装方式可能会在/etc/kubernetes/manifests/etcd.yml中）
```shell
grep -E "key-file|cert-file" /etc/etcd/etcd.config.yml

```
#### 创建ETCD service
首先需要配置Etcd的Service和Endpoint
```yaml
cat > etcd-svc.yaml <<EOF
apiVersion: v1
kind: Endpoints
metadata:
  labels:
    app: etcd-prom
  name: etcd-prom
  namespace: kube-system
subsets:
- addresses:
  - ip: 10.1.1.101
  - ip: 10.1.1.102
  - ip: 10.1.1.103
  ports:
  - name: https-metrics
    port: 2379 # etcd 端口
    protocol: TCP
---
apiVersion: v1
kind: Service
metadata:
  labels:
    app: etcd-prom
  name: etcd-prom
  namespace: kube-system
spec:
  ports:
  - name: https-metrics
    port: 2379
    protocol: TCP
    targetPort: 2379
  type: ClusterIP
EOF
kubectl create -f etcd-svc.yaml
```
#### 通过ClusterIP访问测试：
```shell
curl -s --cert /etc/kubernetes/pki/etcd/etcd.pem \
        --key /etc/kubernetes/pki/etcd/etcd-key.pem \
        https://192.168.18.28:2379/metrics -k | tail -1
```
#### 创建Etcd证书的Secret （证书路径根据实际环境进行更改
```shell
kubectl create secret generic etcd-ssl \
    --from-file=/etc/kubernetes/pki/etcd/etcd-ca.pem \
    --from-file=/etc/kubernetes/pki/etcd/etcd.pem \
    --from-file=/etc/kubernetes/pki/etcd/etcd-key.pem \
    -n monitoring

```
#### 将证书挂载至Prometheus容器
由于Prometheus是Operator部署的，所以只需要修改Prometheus资源即可
```shell
kubectl edit prometheus k8s -n monitoring
```
添加secrets字段
```text
  ruleNamespaceSelector: {}
  ruleSelector: {}
  scrapeInterval: 30s
  secrets:
  - etcd-ssl
  securityContext:
```
保存退出后，Prometheus的Pod会自动重启，重启完成后，查看证书是否挂载（任意一个Prometheus的Pod均可）
```text
[root@master01 ~]# kubectl get po -n monitoring | grep prometheus-k8s-
prometheus-k8s-0                       2/2     Running   0             4m58s
prometheus-k8s-1                       2/2     Running   0             5m15s
[root@master01 ~]# 
[root@master01 ~]# kubectl exec -n monitoring prometheus-k8s-0 -c prometheus -- ls /etc/prometheus/secrets/etcd-ssl/
etcd-ca.pem
etcd-key.pem
etcd.pem
[root@master01 ~]# 
```

#### 创建Etcd ServiceMonitor
```yaml
cat > ETCD-serviceMonitor.yaml <<EOF
apiVersion: monitoring.coreos.com/v1
kind: ServiceMonitor
metadata:
  name: etcd
  namespace: monitoring
  labels:
    app: etcd
spec:
  jobLabel: k8s-app
  endpoints:
  - interval: 30s
    port: https-metrics # 这个 port 对应 Service.spec.ports.name
    scheme: https
    tlsConfig:
      caFile: /etc/prometheus/secrets/etcd-ssl/etcd-ca.pem # 证书路径
      certFile: /etc/prometheus/secrets/etcd-ssl/etcd.pem
      keyFile: /etc/prometheus/secrets/etcd-ssl/etcd-key.pem
      insecureSkipVerify: true # 关闭证书校验
  selector:
    matchLabels:
      app: etcd-prom # 跟svc的lables保持一致
  namespaceSelector:
    matchNames:
    - kube-system
EOF
kubectl apply -f ETCD-serviceMonitor.yaml
```














