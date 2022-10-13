Ingress Controller 安装
### NGINX-ingress
官方安装文档：https://kubernetes.github.io/ingress nginx/deploy/#bare metal clusters
### 将controller的pod的80端口映射至宿主的6868号端口
        ports:
        - containerPort: 80
          name: http
          protocol: TCP
          hostPort: 6868
```shell
kubectl apply -f http://mgmt.ty.com/ingress-controller/nginx-ingress-controller.yaml
```
