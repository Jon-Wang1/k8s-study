接下来创建Istio的Gateway和VirtualService实现域名访问Bookinfo项目。  
首先创建Gateway，假设域名是bookinfo.kubeasy.com。  
Gateway配置如下所示：

```yaml
cat > bookinfo-gateway.yaml <<EOF
apiVersion: networking.istio.io/v1alpha3
kind: Gateway
metadata:
  name: bookinfo-gateway
  namespace: bookinfo  # 可以跨命名空间，namespace写不写不重要
spec:
  selector:
    istio: ingressgateway # 使用默认的 istio ingress gateway
  servers:
  - port:
      number: 80
      name: http
      protocol: HTTP
    hosts:
    - "*.kubeasy.com"
EOF
kubectl apply -f bookinfo-gateway.yaml

```
接下来配置VirtualService，实现对不同微服务的路由：

```yaml
cat > bookinfo-VirtualService.yaml <<EOF
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: bookinfo
  namespace: bookinfo
spec:
  hosts:
  - "bookinfo.kubeasy.com"
  gateways:
  - bookinfo-gateway
  http:
  - match:
    - uri:
        exact: /productpage
    - uri:
        prefix: /static
    - uri:
        exact: /login
    - uri:
        exact: /logout
    - uri:
        prefix: /api/v1/products
    route:
    - destination:
        host: productpage
        port:
          number: 9080
EOF
kubectl apply -f bookinfo-VirtualService.yaml

```