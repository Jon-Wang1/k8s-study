### 使用Filebeat 收集自定义文件日志
#### 创建Kafka和Logstash
首先需要部署Kafka和Logstash至Kubernetes集群，如果企业内已经有比较成熟的技术栈，可以无需部署，直接将Filebeat的输出指向外部Kafka集群即可。
```shell
cd filebeat
helm install zookeeper zookeeper/ -n logging
kubectl get pod -n logging -l app.kubernetes.io/name=zookeeper
helm install kafka kafka/ -n logging
kubectl create -f logstash-service.yaml -f logstash-cm.yaml -f logstash.yaml -n logging
```
#### 需要注意logstash-cm.yaml文件中的一些配置：
```text
➢ input：数据来源，本次示例配置的是Kakfa；
➢ input.kafka.bootstrap_servers：Kafka地址，由于是安装在集群内部的，可以直接使用Kafka集群的Service接口，如果是外部地址，按需配置即可；
➢ input.kafka.topics：Kafka的topic，需要和Filebeat输出的topic一致；
➢ input.kafka.type：定义一个type，可以用于logstash输出至不同的Elasticsearch集群；
➢ output：数据输出至哪里，本次示例输出至Elasticsearch集群，在里面配置了一个判断语句，当type为filebeat-sidecar时，将会输出至Elasticsearch集群，并且index为filebeat-xxx。
```

### 注入Filebeat Sidecar
filebeat-cm说明：
```yaml
      tail_files: true # 从后往前读取
      fields:          # 更加pod的元数据添加分词，便于索引
        pod_name: '${podName}'  
        pod_ip: '${podIp}'
        pod_deploy_name: '${podDeployName}'
        pod_namespace: '${podNamespace}'
```
接下来创建一个模拟程序：
```shell
kubectl create -f filebeat-cm.yaml -n logging
kubectl create -f app-filebeat.yaml -n logging

```

#### 在Kibana中添加filebeat
```text
Kibana上添加Filebeat的索引即可查看日志，添加步骤和EFK一致，只需要更改索引名即可。  
首先点击菜单栏 -> Stack Management
```