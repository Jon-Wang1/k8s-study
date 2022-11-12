### 下载
```shell
wget https://github.com/istio/istio/releases/download/1.13.0/istio-1.13.0-linux-amd64.tar.gz
```
解压后，将Istio的客户端工具istioctl，移动到/usr/local/bin目录下，并安装
```shell
tar xf istio-1.13.0-linux-amd64.tar.gz
cd istio-1.13.0
mv bin/istioctl /usr/local/bin/
istioctl version
istioctl operator init
```

### 检查Istio Operator
```text
[root@master01 ~]# kubectl get pod -n istio-operator
NAME                              READY   STATUS    RESTARTS   AGE
istio-operator-6998666877-5r58s   1/1     Running   0          3m47s
[root@master01 ~]# 
```

### 之后通过定义Istio Operator资源，在Kubernetes中安装Istio
```shell
cat > istio-install.yaml <<EOF
apiVersion: install.istio.io/v1alpha1
kind: IstioOperator
metadata:
  namespace: istio-system
  name: example-istiocontrolplane
spec:
  profile: default  # 根据实际环境修改，https://preliminary.istio.io/latest/docs/setup/additional-setup/config-profiles/
  components: # 自定义组件配置
    ingressGateways: # 自定义 ingressGateway 配置
    - name: istio-ingressgateway
      enabled: true # 开启 ingressGateway
      k8s: # 自定义 ingressGateway 的 Kubernetes 配置
        service: # 将 Service 类型改成 NodePort
          type: NodePort # 默认为load balancer
          ports:
          - port: 15020
            nodePort: 30520
            name: status-port
          - port: 80 # 流量入口 80 端口映射到 NodePort 的 30080 ，之后通过节点IP+30080 即可访问 Istio 服务
            nodePort: 30080
            name: http2
            targetPort: 8080
          - port: 443
            nodePort: 30443
            name: https 
            targetPort: 8443
EOF
istioctl manifest apply -f istio-install.yaml

```

### 安装过程

```text
[root@master01 ~]# istioctl manifest apply -f istio-install.yaml
This will install the Istio 1.13.0 default profile with ["Istio core" "Istiod" "Ingress gateways"] components into the cluster. Proceed? (y/N) y
✔ Istio core installed                                                                                                                                      
✔ Istiod installed                                                                                                                                          
- Processing resources for Ingress gateways. Waiting for Deployment/istio-system/istio-ingressgateway                                                       
- Processing resources for Ingress gateways. Waiting for Deployment/istio-system/istio-ingressgateway                                                       
- Processing resources for Ingress gateways. Waiting for Deployment/istio-system/istio-ingressgateway                                                       

✔ Ingress gateways installed                                                                                                                                
✔ Installation complete                                                                                                                                     Making this installation the default for injection and validation.

Thank you for installing Istio 1.13.  Please take a few minutes to tell us about your install/upgrade experience!  https://forms.gle/pzWZpAvMVBecaQ9h9
[root@master01 ~]# 
```

### 测试

接下来创建一个测试的Namespace，并添加一个istio-injection=enabled的标签，之后在该Namespace下创建的Pod就会被自动注入Istio的Proxy。
#### 创建Namespace并添加Label

```shell
kubectl create ns istio-test
kubectl label namespace istio-test istio-injection=enabled
```

切换目录至istio的安装包，然后创建测试应用，此时创建的Pod会被自动注入一个istio-proxy的容器
```shell
kubectl apply -f samples/sleep/sleep.yaml -n istio-test
kubectl get pod -n istio-test
```