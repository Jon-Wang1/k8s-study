非云原生监控Exporter
#### 部署测试用例
```shell
kubectl create deploy mysql --image=mysql:5.7.23
kubectl set env deploy/mysql MYSQL_ROOT_PASSWORD=mysql
kubectl expose deploy mysql --port 3306
kubectl get svc -l app=mysql
```
登录MySQL，创建Exporter所需的用户和权限（如果已经有需要监控的 MySQL，可以直接执行此步骤即可）。
```shell
kubectl exec -ti mysql-69d6f69557 5vnvg bash
mysql -uroot -pmysql
CREATE USER 'exporter'@'%' IDENTIFIED BY 'exporter' WITH MAX_USER_CONNECTIONS 3;
GRANT PROCESS, REPLICATION CLIENT, SELECT ON *.* TO 'exporter'@'%';
quit
```
#### 配置MySQL Exporter采集MySQL监控数据
##### 创建Exporter
```yaml
cat > mysql-exporter.yaml <<EOF
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mysql-exporter
  namespace: monitoring
spec:
  replicas: 1
  selector:
    matchLabels:
      k8s-app: mysql-exporter
  template:
    metadata:
      labels:
        k8s-app: mysql-exporter
    spec:
      containers:
      - name: mysql-exporter
        image: bitnami/mysqld-exporter
        env:
        - name: DATA_SOURCE_NAME
          value: "exporter:exporter@(mysql.default:3306)/"
        imagePullPolicy: IfNotPresent
        ports:
        - containerPort: 9104
---
apiVersion: v1
kind: Service
metadata:
  name: mysql-exporter
  namespace: monitoring
  labels:
    k8s-app: mysql-exporter
spec:
  type: ClusterIP
  selector:
    k8s-app: mysql-exporter
  ports:
  - name: api
    port: 9104
    protocol: TCP
EOF
kubectl create -f mysql-exporter.yaml
kubectl get -f mysql-exporter.yaml

```
通过该Service地址，检查是否能正常获取Metrics数据  
```shell
curl 192.168.97.37:9104/metrics | tail -1
```
#### 配置相应的ServiceMonitor
```yaml
cat > mysql-sm.yaml <<EOF
apiVersion: monitoring.coreos.com/v1
kind: ServiceMonitor
metadata:
  name: mysql-exporter
  namespace: monitoring
  labels:
    k8s-app: mysql-exporter
    namespace: monitoring
spec:
  jobLabel: k8s-app
  endpoints:
  - port: api
    interval: 30s
    scheme: http
  selector:
    matchLabels:
      k8s-app: mysql-exporter
  namespaceSelector:
    matchNames:
    - monitoring
EOF
kubectl apply -f mysql-sm.yaml
```
1. 接下来在Prometheus的Web UI看该exporter
2. 在grafana中添加图表：https://grafana.com/grafana/dashboards/6239-mysql/





