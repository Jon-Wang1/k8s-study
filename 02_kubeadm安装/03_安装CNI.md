## 安装calico

### 以下步骤只在master01执行
```shell
cd ~
git clone https://github.com/dotbalo/k8s-ha-install.git
cd /root/k8s-ha-install 
git checkout manual-installation-v1.25.x 
cd calico/

```

	
### 修改calico配置文件Pod网段：
```shell
cd /root/k8s-ha-install/calico/
# 将k8s集群配置文件中的cluster-cidr赋值给POD_SUBNET变量
POD_SUBNET=`cat /etc/kubernetes/manifests/kube-controller-manager.yaml | grep cluster-cidr= | awk -F= '{print $NF}'`
# 使用POD_SUBNET变量替换calico配置文件中的POD_CIDR字段
sed -i "s#POD_CIDR#${POD_SUBNET}#g" calico.yaml
kubectl apply -f calico.yaml

```
