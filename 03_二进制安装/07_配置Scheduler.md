## Scheduler
### 所有Master节点配置kube-scheduler service（所有master节点配置一样）
```shell
cat > /usr/lib/systemd/system/kube-scheduler.service <<EOF 
[Unit]
Description=Kubernetes Scheduler
Documentation=https://github.com/kubernetes/kubernetes
After=network.target

[Service]
ExecStart=/usr/local/bin/kube-scheduler \
      --v=2 \
      --logtostderr=true \
      --leader-elect=true \
      --authentication-kubeconfig=/etc/kubernetes/scheduler.kubeconfig \
      --authorization-kubeconfig=/etc/kubernetes/scheduler.kubeconfig \
      --kubeconfig=/etc/kubernetes/scheduler.kubeconfig

Restart=always
RestartSec=10s

[Install]
WantedBy=multi-user.target
EOF

systemctl daemon-reload
systemctl enable --now kube-scheduler

systemctl status kube-scheduler
```
 
