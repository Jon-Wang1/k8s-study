### 使用EFK收集控制台日志
收集的日志包括所有pod的控制台日志和主机K8S相关组件的日志，配置信息在fluentd-es-configmap.yaml文件中。  
注意：生产环境部署时，要修改es的持久卷。
#### 部署EFK
```shell
mkdir efk
cd efk
git clone https://github.com/Jon-Wang1/k8s-dukuan.git
cd k8s/efk-7.10.2/
kubectl create -f create-logging-namespace.yaml
kubectl create -f es-service.yaml
kubectl create -f es-statefulset.yaml
kubectl apply -f kibana-deployment.yaml
kubectl apply -f kibana-service.yaml 
kubectl label node node01.ty.com node02.ty.com node03.ty.com fluentd=true
kubectl apply -f fluentd-es-configmap.yaml -f fluentd-es-ds.yaml
```

#### 使用kibana
```text
接下来查看Kibana暴露的端口号，访问Kibana：
kubectl get svc -n logging
使用宿主机的IP和端口访问kibana
http://10.1.1.202:32641/kibana
点击Explore on my own，之后再点击Visualize
之后点击Add your data → Create index pattern
在Index pattern name输入索引名logstash*，然后点击Next Step
再次选择timestamp，即可创建索引：
之后点击菜单栏->Discover即可看到相关日志
```
