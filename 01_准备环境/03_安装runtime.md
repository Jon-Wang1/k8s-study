### CentOS 7安装yum源如下：
```` shell
curl -o /etc/yum.repos.d/CentOS-Base.repo https://mirrors.aliyun.com/repo/Centos-7.repo
yum install -y yum-utils device-mapper-persistent-data lvm2
yum-config-manager --add-repo https://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
cat <<EOF > /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64/
enabled=1
gpgcheck=0
repo_gpgcheck=0
gpgkey=https://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg https://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg
EOF
sed -i -e '/mirrors.cloud.aliyuncs.com/d' -e '/mirrors.aliyuncs.com/d' /etc/yum.repos.d/CentOS-Base.repo

````

### 所有节点安装docker-ce-20.10：
``` shell
yum install docker-ce-20.10.* docker-ce-cli-20.10.* -y

```
无需启动Docker，只需要配置和启动Containerd即可。
### 首先配置Containerd所需的模块（所有节点）：
``` shell
cat <<EOF | sudo tee /etc/modules-load.d/containerd.conf
overlay
br_netfilter
EOF

# 所有节点加载模块：
modprobe -- overlay
modprobe -- br_netfilter
```
### 所有节点，配置Containerd所需的内核：
``` shell
cat <<EOF | sudo tee /etc/sysctl.d/99-kubernetes-cri.conf
net.bridge.bridge-nf-call-iptables  = 1
net.ipv4.ip_forward                 = 1
net.bridge.bridge-nf-call-ip6tables = 1
EOF

# 所有节点加载内核：
sysctl --system

```
### 所有节点配置Containerd的配置文件：
``` shell
mkdir -p /etc/containerd
containerd config default | tee /etc/containerd/config.toml

```

### 所有节点将Containerd的Cgroup改为Systemd：
vim /etc/containerd/config.toml
找到containerd.runtimes.runc.options，添加SystemdCgroup = true（如果已存在直接修改，否则会报错），如下图所示：
![img.png](img.png)
### 所有节点将sandbox_image的Pause镜像
改成符合自己版本的地址registry.cn-hangzhou.aliyuncs.com/google_containers/pause:3.6：

### 所有节点启动Containerd，并配置开机自启动：
``` shell
systemctl daemon-reload
systemctl enable --now containerd

```
### 所有节点配置crictl客户端连接的运行时位置：
``` shell
cat > /etc/crictl.yaml <<EOF
runtime-endpoint: unix:///run/containerd/containerd.sock
image-endpoint: unix:///run/containerd/containerd.sock
timeout: 10
debug: false
EOF

```