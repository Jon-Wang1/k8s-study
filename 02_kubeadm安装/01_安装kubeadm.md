### 首先在Master01节点查看最新的Kubernetes版本是多少：
```` shell
yum list kubeadm.x86_64 --showduplicates | sort -r

````

### 所有节点安装1.25最新版本kubeadm、kubelet和kubectl：
```` shell
yum install kubeadm-1.25* kubelet-1.25* kubectl-1.25* -y
# 如果选择的是Containerd作为的Runtime，需要更改Kubelet的配置使用Containerd作为Runtime：
cat >/etc/sysconfig/kubelet<<EOF
KUBELET_KUBEADM_ARGS="--container-runtime=remote --runtime-request-timeout=15m --container-runtime-endpoint=unix:///run/containerd/containerd.sock"
EOF
````

### 所有节点设置Kubelet开机自启动
（由于还未初始化，没有kubelet的配置文件，此时kubelet无法启动，无需管理）：
```` shell
systemctl daemon-reload
systemctl enable --now kubelet
# 此时kubelet是起不来的，日志会有报错不影响！
````