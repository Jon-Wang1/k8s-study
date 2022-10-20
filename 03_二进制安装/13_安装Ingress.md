```shell
helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
helm pull ingress-nginx/ingress-nginx --version 4.3.0
tar xf ingress-nginx-4.3.0.tgz
cd  ingress-nginx/
```
### 需要修改vim values.yaml
1. Controller和admissionWebhook的镜像地址
2. 镜像的digest值注释
3. hostNetwork设置为true
4. dnsPolicy设置为 ClusterFirstWithHostNet
5. nodeSelector添加ingress: "true"部署至指定节点
6. 类型更改为kind: DaemonSet
7. 将ingressClassResource设置为默认的ingressClass
8. 部署ingress，给需要部署ingress的节点上打标签

```shell
kubectl label node node01.ty.com ingress=true
kubectl create ns ingress-nginx
helm install ingress-nginx -n ingress-nginx .
```

### 入门使用
#### 创建web服务器和service
```shell
kubectl create ns study-ingress
kubectl create deploy nginx --image=nginx -n study-ingress
kubectl expose deploy nginx --port 80 -n study-ingress
```
#### 创建ingress
```yaml
cat > web-ingress.yaml <<EOF
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: web1-ingress
  namespace: study-ingress
spec:
  rules:
  - host: web1.ty.com
    http:
      paths:
      - backend:
          service:
            name: nginx
            port:
              number: 80
        path: /
        pathType: ImplementationSpecific
EOF
kubectl apply -f web-ingress.yaml
```

### Ingress Nginx 域名重定向 Redirect
```yaml
cat > redirect.yaml <<EOF
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  annotations:
    nginx.ingress.kubernetes.io/permanent-redirect: https://www.baidu.com
  name: nginx-redirect
  namespace: study-ingress
spec:
  rules:
  - host: redirect.ty.com
    http:
      paths:
      - backend:
          service:
            name: nginx
            port:
              number: 80
        path: /
        pathType: ImplementationSpecific
EOF
kubectl apply -f redirect.yaml
```

### Ingress Nginx 前后端分离 Rewrite
创建一个应用模拟后端服务：
```shell
kubectl create deploy backend-api --image=harbor.ty.com/public/ty_lb -n study-ingress
```
创建Service暴露该应用:
```shell
kubectl expose deploy backend-api --port 80 --target-port=5000 -n study-ingress
```
通过Ingress Nginx的Rewrite功能，将/api-a重写为“/”，配置示例如下:
```yaml
cat > rewrite.yaml <<EOF
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /$2
  name: backend-api
  namespace: study-ingress
spec:
  rules:
  - host: rewrite.ty.com
    http:
      paths:
      - backend:
          service:
            name: backend-api
            port:
              number: 80
        path: /api-a(/|$)(.*)
        pathType: ImplementationSpecific
EOF
kubectl apply -f rewrite.yaml
```

### 错误代码重定向
通过 Helm 进行更改修改values.yaml 如下所示位置：
```yaml
## Default 404 backend
##
defaultBackend:
  ##
  enabled: true
```
更新 ConfigMap，在template目录下更改controller-configmap.yaml，在data下添加下面的数据：
```yaml
  apiVersion: v1
  client_max_body_size: 20m
  custom http errors: "404,415,503"
```
更新release：
```shell
helm upgrade ingress-nginx -n ingress-nginx .
```

### Ingress Nginx SSL
```shell
openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout tls.key -out tls.crt -subj "/CN=web1.ty.com"
kubectl create secret tls ca-secret --cert=tls.crt --key=tls.key -n study-ingress
```

配置Ingress 添加 TLS 配置
```yaml
cat > ingress-ssl.yaml <<EOF
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: web1-ingress
  namespace: study-ingress
spec:
  ingressClassName: nginx # for k8s >= 1.22
  rules:
  - host: web1.ty.com
    http:
      paths:
      - backend:
          service:
            name: nginx
            port:
              number: 80
        path: /
        pathType: ImplementationSpecific
  tls:
  - hosts:
    - web1.ty.com
    secretName: ca-secret
EOF
kubectl apply -f ingress-ssl.yaml
```

### Ingress Nginx 匹配请求头
```shell
#首先部署移动端应用：
kubectl create deploy phone --image= -n study-ingress
kubectl expose deploy phone --port 80 -n study-ingress
kubectl create ingress phone --rule=m.ty.com/*=phone:80 -n study-ingress
#部署PC端应用：
kubectl create deploy pc --image= -n study-ingress
kubectl expose deploy pc --port 80 -n study-ingress
```
```text
创建PC端ingress：
注意 Ingress annotations 的 nginx.ingress.kubernetes.io/server-snippet 配置。
Snippet 配置是专门用于一些复杂的Nginx配置，和Nginx配置通用。
```
```yaml
cat > pc-ingress.yaml <<EOF
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  annotations:
    kubernetes.io/ingress.class: nginx
    nginx.ingress.kubernetes.io/server-snippet: |
      set \$agentflag 0;
            if (\$http_user_agent ~* "(Android|iPhone|Windows Phone|UC|Kindle)" ){
               set \$agentflag 1;
               }
            if ( \$agentflag = 1 ) {
               return 301 http://m.ty.com;
               }
  name: pc
  namespace: study-ingress
spec:
  rules:
  - host: "pc-m.ty.com"
    http:
      paths:
      - backend:
          service:
            name: nginx
            port:
              number: 80
        path: /
        pathType: ImplementationSpecific
EOF
kubectl apply -f pc-ingress.yaml
```

### Ingress Nginx 基本认证
有些网站可能需要通过密码来访问，对于这类网站可以使用Nginx的basic-auth设置密码访问。
具体方法如下，由于需要使用htpasswd工具，所以需要安装 httpd： 
```shell
yum install httpd -y
#使用htpasswd创建foo用户的密码：
htpasswd -c auth foo
Cisc0123
Cisc0123
cat auth 
#基于之前创建的密码文件创建Secret：
kubectl create secret generic basic-auth --from-file=auth -n study-ingress 
```

```yaml
cat > with-auth-ingress.yaml <<EOF
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  annotations:
    nginx.ingress.kubernetes.io/auth-realm: Please Input Your Username and Password
    nginx.ingress.kubernetes.io/auth-secret: basic-auth
    nginx.ingress.kubernetes.io/auth-type: basic
  name: with-auth-ingress
  namespace: study-ingress
spec:
  rules:
  - host: "auth.ty.com"
    http:
      paths:
      - backend:
          service:
            name: nginx
            port:
              number: 80
        path: /
        pathType: ImplementationSpecific
EOF
kubectl apply -f with-auth-ingress.yaml
```

### Ingress Nginx 速率限制
有时候可能需要限制速率以降低后端压力，或者限制单个IP每秒的访问速率防止攻击。此时可以使用Nginx的rate limit进行配置。  
首先没有加速率限制，使用ab进行访问，Failed为0:  
```shell
ab -c 10 -n 100 http://m.ty.com/ | grep requests
```

添加速率限制，限制只能有一个连接，只需要添加nginx.ingress.kubernetes.io/limit-connections为1即可： 
```yaml
cat > rate-limit-ingress.yaml <<EOF
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  annotations:
    nginx.ingress.kubernetes.io/limit-connections: "1"
  name: phone
  namespace: study-ingress
spec:
  rules:
  - host: m.ty.com
    http:
      paths:
      - backend:
          service:
            name: nginx
            port:
              number: 80
        path: /
        pathType: ImplementationSpecific
EOF
kubectl apply -f rate-limit-ingress.yaml
```
```text
还有很多其它方面的限制，常用的配置如下：
限制每秒的连接，单个IP:
nginx.ingress.kubernetes.io/limit-rps
限制每分钟的连接，单个IP:
nginx.ingress.kubernetes.io/limit-rpm
限制客户端每秒传输的字节数 单位为K,需要开启 proxy-buffering:
nginx.ingress.kubernetes.io/limit-rate
速率限制白名单:
nginx.ingress.kubernetes.io/limit whitelist
```


### 使用Nginx实现灰度/金丝雀发布
##### 创建V1版本
首先创建模拟Production（生产）环境的Namespace和服务： 
```shell
kubectl create ns production
kubectl create deploy canary-v1 --image=harbor.ty.com/public/ty_lb -n production
kubectl expose deploy canary-v1 --port 8080 --target-port=5000 -n production
kubectl create ingress canary-v1 --rule=canary.ty.com/*=canary-v1:8080 -n production
```

##### 接下来创建v2版本，充当灰度环境
```shell
kubectl create ns canary
kubectl create deploy canary-v2 --image=nginx -n canary
kubectl expose deploy canary-v2 --port 80 -n canary
```

#### Canary版本切入部分流量
创建v2版本的Ingress时，需要添加两个注释，一个是nginx.ingress.kubernetes.io/canary，表明是灰度环境。  
nginx.ingress.kubernetes.io/canary-weight表明切多少流量到该环境，本示例为10%：  
```yaml
cat > canary-v2.yaml <<EOF
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  annotations:
    nginx.ingress.kubernetes.io/canary: "true"
    nginx.ingress.kubernetes.io/canary-weight: "50"
  name: canary-v2
  namespace: canary
spec:
  rules:
  - host: canary.ty.com
    http:
      paths:
      - backend:
          service:
            name: canary-v2
            port:
              number: 80
        path: /
        pathType: ImplementationSpecific
EOF
kubectl apply -f canary-v2.yaml
```

#### 测试灰度发布
接下来使用Ruby脚本进行测试，此脚本会输出v1和v2的访问次数比值：
使用vi编辑
vi  test-canary.rb
```shell
counts = Hash.new(0)

100.times do
  output = `curl -s canary.ty.com | grep 'Canary' | awk '{print $2}' | awk -F"<" '{print $1}'`
  counts[output.strip.split.last] += 1
end

puts counts
```
安装ruby并测试
```shell
yum install ruby -y
ruby test-canary.rb
```