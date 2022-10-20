https://prometheus.io/docs/alerting/latest/configuration/
https://github.com/dotbalo/k8s/blob/master/prometheus-operator/alertmanager.yaml
https://github.com/prometheus/alertmanager/blob/main/doc/examples/simple.yml

### 配置邮件通知
在/root/prometheus/kube-prometheus/manifests找到alertmanager-secret.yaml  
添加下面的信息
```yaml
stringData:
  alertmanager.yaml: |-
    "global":
      "resolve_timeout": "5m"
      smtp_smarthost: 'smtp.qq.com:465'
      smtp_from: '250518824@qq.com'
      smtp_auth_username: '250518824@qq.com'
      smtp_auth_password: 'egmsspcgthhubihb'
      smtp_require_tls: false
……
    "receivers":
    - "name": "Default"
      email_configs:
      - to: 'qzw091736@icloud.com'
        send_resolved: true
……
    "route":
      "group_by":
      - "namespace"
      - job
      - alertname
      "group_interval": "5m"
```

### 配置企业微信告警
ww0aaba912a4eac1d2
partID  2
AgentId 1000002
secret yk5b_eR26M-RR8pVLf5GE6mO0TMvLnRcOrDjweVkgUo

```yaml
stringData:
  alertmanager.yaml: |-
    "global":
      "resolve_timeout": "5m"
      smtp_smarthost: 'smtp.qq.com:465'
      smtp_from: '250518824@qq.com'
      smtp_auth_username: '250518824@qq.com'
      smtp_auth_password: 'egmsspcgthhubihb'
      smtp_require_tls: false
      wechat_api_url: "https://qyapi.weixin.qq.com/cgi-bin/"
      wechat_api_corp_id: "ww0aaba912a4eac1d2"
    "inhibit_rules":
    - "equal":
……

    "receivers":
    - name: wechat-ops
      wechat_configs:
      - send_resolved: true
        to_party: 2  # 部门ID
        to_user: '@all'  # 发给指定人或所有人
        agent_id: 1000002
        api_secret: "yk5b_eR26M-RR8pVLf5GE6mO0TMvLnRcOrDjweVkgUo"
……
      "routes":
      - "matchers":
        - "alertname = Watchdog"
        "receiver": "wechat-ops"
```

### PrometheusRule
kubectl get prometheusrule -n monitoring
Prometheus通过PrometheusRule中的labels信息找到该Rule
```yaml
apiVersion: monitoring.coreos.com/v1
kind: PrometheusRule
metadata:
  creationTimestamp: "2022-10-19T13:04:34Z"
  generation: 1
  labels:
    app.kubernetes.io/component: alert-router
    app.kubernetes.io/instance: main
    app.kubernetes.io/name: alertmanager
    app.kubernetes.io/part-of: kube-prometheus
    app.kubernetes.io/version: 0.24.0
    prometheus: k8s
    role: alert-rules
  name: alertmanager-main-rules
```

### 配置告警规则
假设需要对域名访问延迟进行监控，访问延迟大于1秒进行告警，此时可以创建一个PrometheusRule如下:   
vim blackbox-AlertRule.yaml 
```yaml
apiVersion: monitoring.coreos.com/v1
kind: PrometheusRule
metadata:
  labels:
    app.kubernetes.io/component: exporter
    app.kubernetes.io/name: blackbox-exporter
    prometheus: k8s
    role: alert-rules
  name: blackbox
  namespace: monitoring
spec:
  groups:
  - name: blackbox-exporter
    rules:
    - alert: DomainAccessDelayExceeds1s
      annotations:
        description: 域名：{{ $labels.instance }} 探测延迟大于1秒,当前延迟为:{{ $value }}
        summary: 域名探测，访问延迟超过 1 秒
      expr: sum(probe_http_duration_seconds{job=~"blackbox"}) by (instance) > 1
      for: 1m
      labels:
        severity: warning
        type: blackbox
```
 kubectl create -f blackbox-AlertRule.yaml
#### 在告警配置中添加
```yaml
      "routes":
      - match:
          type: blackbox
        receiver: "Default"
        repeat_interval: 10m
```
























