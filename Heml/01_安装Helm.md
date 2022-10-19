使用Helm 创建 Kafka 、 Zookeeper 集群
#### 客户端安装： 
https://helm.sh/docs/intro/install/
```shell
wget https://get.helm.sh/helm-v3.10.1-linux-amd64.tar.gz
tar -zxvf helm-v3.10.1-linux-amd64.tar.gz
mv linux-amd64/helm /usr/local/bin/helm
```
#### 添加添加仓库
bitnami和官方helm仓库：   
```shell
helm repo add bitnami https://charts.bitnami.com/bitnami
helm repo add stable https://charts.helm.sh/stable
```

#### 安装方式一：先下载后安装
```shell
helm pull bitnami/zookeeper
# 修改values.yaml 相应配置：副本数、 auth 、持久化
helm install -n public-service zookeeper .
```
#### 安装方式二：直接安装
```shell
helm install kafka bitnami/kafka \
   --set zookeeper.enabled=false \
   --set replicaCount=3 \
   --set externalZookeeper.servers=zookeeper \
   --set persistence.enabled=false \
   -n public-service

```

### kafka验证
```shell
kubectl run kafka-client \
   --restart='Never' \
   --image=docker.io/bitnami/kafka:2.8.0-debian-10-r30 \
   --namespace public-service \
   --command -- sleep infinity
   
kubectl exec -it kafka-client --namespace public-service -- bash
```
```shell
kafka-console-producer.sh \
   --broker-list kafka-0.kafka-headless.public-service.svc.cluster.local:9092,\
kafka-1.kafka-headless.public-service.svc.cluster.local:9092,\
kafka-2.kafka-headless.public-service.svc.cluster.local:9092 \
   --topic test
   
   
kafka-console-consumer.sh \
   --bootstrap-server kafka.public-service.svc.cluster.local:9092 \
   --topic test \
   --from-beginning
```
